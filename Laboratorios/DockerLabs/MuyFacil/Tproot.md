# Tproot

### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Realizamos un escaneo inicial de puertos en la máquina objetivo para identificar servicios activos:

![[Pasted image 20260618225121.png]]

**Servicios identificados:**

* **Puerto 21/TCP (FTP):** Servicio de transferencia de archivos activo. Ver teoría en [FTP.md](<../../../Pentesting Notes/1_Enumeration/FTP.md>).
* **Puerto 80/TCP (HTTP):** Servidor web activo. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).

***

### Enumeración de Servicios y Vulnerabilidades

#### Análisis del Servicio FTP

Inspeccionamos la versión del servicio FTP que está corriendo en el servidor. Al ser una versión antigua, procedemos a verificar si cuenta con vulnerabilidades públicas explotables:

![[Pasted image 20260618225158.png]]

* **Resultado:** \[Explicar aquí el exploit o vulnerabilidad identificada para la versión de FTP y cómo se procedió a explotarlo].

***

### Fase de Explotación / Intrusión

* \[Documentar el método de ejecución del exploit, las opciones configuradas y cómo se obtuvo el acceso inicial a la máquina].

***

### Escalada de Privilegios

* \[Documentar los comandos de enumeración ejecutados en el sistema víctima y los pasos realizados para obtener privilegios de root].

***

### Relaciones y Conceptos

* **Teoría:** [FTP.md](<../../../Pentesting Notes/1_Enumeration/FTP.md>), [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>)
* **Laboratorios Relacionados:** [ApiBase](../Facil/ApiBase.md) (Comparte análisis relacionado con servicios FTP)



**notas que se borraron:**

Hacemos el escaneo incial:![[Pasted image 20260618225121.png]]Vemos que solo hay el puerto 80 abierto HTTP y el 21 FTP.Vemos la version de FTP que es una antigua y comprobamos y tiene un exploit:![[Pasted image 20260618225158.png]]
Collapse file‎Laboratorios/DockerLabs/MuyFacil/Trust.md‎Copy file name to clipboard+54Lines changed: 54 additions & 0 deletionsDisplay the source diffDisplay the rich diffOriginal file line numberDiff line numberDiff line change@@ -0,0 +1,54 @@Empzamos el escaneo nmap:![[Pasted image 20260624125557.png]]Vemos que hay el puerto 22 SSH abierto y el puerto 80 HTTP abierto.SI intentamos acceder a la web, vemos que esta la pagina por defecto de apache, pero si hacemos fuzzing de directorios y extensiones con el comando:```bashgobuster dir -u 'http://172.17.0.2' -w /usr/share/wordlist/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -x php,html,txt -t 50 --exclude-length 10701```![[Pasted image 20260624125713.png]]Encontramos un archivo php llamado *secret.php*. Intentamos acceder a él:![[Pasted image 20260624125858.png]]Nos sale solo ese texto, asique intentamos mirar las peticiones que hace GET pero no encontramos nada. Si hacemos fuerza bruta al servidor SSH con ese nombre *"mario"* con el comando **hydra**:``` bashhydra -l mario -P /usr/share/wordlist/Rockyou.txt -t 50 -I ssh://172.17.0.2```![[Pasted image 20260624130031.png]]Encontramos credenciales válidas, asi que accedemos al server SSH:![[Pasted image 20260624130050.png]]Pero no somos root, asi que vemos si podemos escalar privilegios:SI ejecutamos *sudo -l* para ver que comandos podemos ejecutar como sudo y que permisos.![[Pasted image 20260624130232.png]]Vemos que vim se puede ejecutar como usuario root siendo mario.Asi que si entramos a vim, y escribimos bash, nos dará una shell como root:``` bashsudo vim:!bash```![[Pasted image 20260624130345.png]]