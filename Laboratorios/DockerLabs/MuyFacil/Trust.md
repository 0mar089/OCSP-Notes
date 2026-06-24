
Empzamos el escaneo nmap:

![[Pasted image 20260624125557.png]]

Vemos que hay el puerto 22 SSH abierto y el puerto 80 HTTP abierto.

SI intentamos acceder a la web, vemos que esta la pagina por defecto de apache, pero si hacemos fuzzing de directorios y extensiones con el comando:

```bash
gobuster dir -u 'http://172.17.0.2' -w /usr/share/wordlist/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -x php,html,txt -t 50 --exclude-length 10701

```

![[Pasted image 20260624125713.png]]

Encontramos un archivo php llamado *secret.php*. Intentamos acceder a él:

![[Pasted image 20260624125858.png]]

Nos sale solo ese texto, asique intentamos mirar las peticiones que hace GET pero no encontramos nada. 

Si hacemos fuerza bruta al servidor SSH con ese nombre *"mario"* con el comando **hydra**:

``` bash
hydra -l mario -P /usr/share/wordlist/Rockyou.txt -t 50 -I ssh://172.17.0.2
```

![[Pasted image 20260624130031.png]]

Encontramos credenciales válidas, asi que accedemos al server SSH:

![[Pasted image 20260624130050.png]]

Pero no somos root, asi que vemos si podemos escalar privilegios:

SI ejecutamos *sudo -l* para ver que comandos podemos ejecutar como sudo y que permisos.

![[Pasted image 20260624130232.png]]

Vemos que vim se puede ejecutar como usuario root siendo mario.

Asi que si entramos a vim, y escribimos bash, nos dará una shell como root:

``` bash
sudo vim
:!bash
```

![[Pasted image 20260624130345.png]]




