
Empezamos con un escaneo nmap:

![[Pasted image 20260630232140.png]]

Encontramos el puerto 80 HTTP y el puerto 22 HTTP.

Hacemos un whatweb antes para ver las tecnologias que hay:

![[Pasted image 20260630232240.png]]


Y encontramos la pagina por defecto de Apache.

![[Pasted image 20260630232308.png]]


Intentamos ahcer un ffuf, para buscar otros directorios posibles, y encontramos el index.php:

![[Pasted image 20260630232934.png]]

![[Pasted image 20260630232951.png]]

Y esta misma pagina web nos indica que hay otro archivo secreto (supuestamente una contraseña)

![[Pasted image 20260630233000.png]]

Ahora como no tenemos otra cosa que hacer, intentamos hacer fuzzing a la pagina web y encontramos que un parametro en la petición *GET* por url, es `page`: 

```
http://172.17.0.2/index.php?page=xxxxxx
```

![[Pasted image 20260630235046.png]]


Probamos a acceder a un archivo y efectivamente, es vulnerable a path traversal. 

![[Pasted image 20260630235021.png]]


Asi que vemos que hay dos usuarios posibles en el sistema *pinguino* y *mario*. Intentamos hacer fuerza bruta para encontrar las contraseñas pero nada. 

![[Pasted image 20260630235159.png]]

Pero luego estaba el .hidden_pass que hemos encontrado antes y probando con uno de los dos usuarios, las credenciales eran:

```
pinguino:balu
```

nos metemos y vemos una nota:

![[Pasted image 20260630235302.png]]

entonces hacemos ssh con el usuario de mario:

![[Pasted image 20260630235418.png]]

Vemos que mario no puede ejecutar archivos con privilegio de root. Y tampoco hay algo interesante en las carpetas de la pagina web.

Asi que listamos por permisos *SUID*:

![[Pasted image 20260630235625.png]]

Y vemos que python3.8 se puede ejecutar, asi que explotamos esa vulnerabilidad. AL mirar en GTFObins, encontramos un one liner que permite elevar los privilegios:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'
```



