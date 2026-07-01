## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos en el host objetivo para identificar los servicios activos:

![[Escaneo.png]]

**Servicios identificados:**
* **Puerto 21/TCP (FTP):** Servicio FTP activo con la opción de ingreso anónimo (anonymous login) habilitada. Ver teoría en [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]].
* **Puerto 22/TCP (SSH):** Servicio de shell segura (OpenSSH) para administración remota. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web Apache. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración de Servicios y Web

### Enumeración del FTP Anónimo
Accedemos al servicio FTP utilizando las credenciales por defecto (anonymous:anonymous) para verificar si hay archivos expuestos:

![[Pasted image 20260214123231.png]]

* **Hallazgo:** Del contenido descargado o visualizado en el FTP, logramos extraer posibles nombres de usuarios del sistema: Gonza, Russoski y Nágore.

### Reconocimiento Web (WhatWeb)
Antes de interactuar directamente con la interfaz del servidor web, analizamos las tecnologías empleadas mediante whatweb:

![[Pasted image 20260214123429.png]]

El sitio web no parece utilizar tecnologías complejas o de riesgo evidente.

### Inspección Web
Accedemos a la página web en el puerto 80:

![[Pasted image 20260214123546.png]]

Vemos una página informativa de un gimnasio. Si bajamos en la página, hay un formulario para solicitar información o ayuda:

![[Pasted image 20260214123619.png]]

Al completar e interactuar con el formulario, este genera una petición POST enviando los datos correspondientes:

![[Pasted image 20260214123717.png]]

---

## Descubrimiento de Directorios (Fuzzing)

Para encontrar archivos o rutas ocultas en el servidor web, realizamos fuerza bruta de directorios:

![[Pasted image 20260214123809.png]]

Identificamos dos rutas interesantes que devuelven un código de redirección: /important y /backup. Procedemos a examinarlas:

### Ruta /important
Accedemos al directorio y encontramos un archivo llamado important.md:

![[Pasted image 20260214123942.png]]

Al abrir el archivo important.md, contiene información poco relevante o escasa:

![[Pasted image 20260214123926.png]]

### Ruta /backup
Accedemos al directorio de backups:

![[Pasted image 20260214124022.png]]

Encontramos un archivo de respaldo que nos brinda información crítica:

![[Pasted image 20260214124033.png]]

* **Hallazgo Clave:** Confirmamos que russoski es un usuario válido en el sistema y se menciona que aún no ha cambiado su contraseña.

---

## Fase de Explotación / Intrusión

### Fuerza Bruta al SSH (Medusa)
Con el usuario confirmado (russoski), procedemos a realizar un ataque de fuerza bruta contra el servicio SSH utilizando medusa y el diccionario estándar rockyou.txt:

```bash
sudo medusa -M ssh -h 172.17.0.2 -u russoski -P /usr/share/wordlists/rockyou.txt -t 10
```

![[Pasted image 20260214131053.png]]

* **Credencial encontrada:** russoski:password (verificada en el resultado de medusa).
* **Acceso:** Establecemos conexión SSH con éxito y obtenemos acceso al servidor con privilegios bajos.

---

## Escalada de Privilegios

### Enumeración Interna
Una vez dentro como el usuario russoski, listamos el contenido de su directorio personal:

![[Pasted image 20260214133103.png]]

No se observa información confidencial o vectores inmediatos aquí. Procedemos a revisar los privilegios de sudo asignados al usuario:

```bash
sudo -l
```

![[Pasted image 20260214133206.png]]

* **Vulnerabilidad de Configuración:** El usuario russoski puede ejecutar el editor de texto /usr/bin/vim como root sin proporcionar contraseña (NOPASSWD). Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].

### Explotación de Sudoers (Vim)
Podemos aprovecharnos de los privilegios de ejecución de vim para lanzar una shell interactiva con permisos de superusuario (root):

Ejecutamos:
```bash
sudo vim
```

Dentro del editor, ejecutamos el comando de escape para llamar a la shell:
```text
:!bash
```

¡Obtenemos una shell interactiva como el usuario root con control total sobre el sistema!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/MuyFacil/Trust\|Trust]] (Comparte escalada por vim)
