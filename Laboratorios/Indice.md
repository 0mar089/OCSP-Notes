# Indice General de Laboratorios

## DockerLabs

### Muy Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [Obsession](../Laboratorios/DockerLabs/MuyFacil/Obsession.md) | FTP Anonimo, Fuzzing Web (`/backup`) y SSH brute force | Sudoers (Vim) | [FTP.md](../Pentesting%20Notes/1_Enumeration/FTP.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Trust](../Laboratorios/DockerLabs/MuyFacil/Trust.md) | Fuzzing Web (`secret.php`) y SSH brute force | Sudoers (Vim) | [HTTP & HTTPS.md](../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Hedgehog](../Laboratorios/DockerLabs/MuyFacil/Hedgehog.md) | Inversion de diccionario (Rockyou) y SSH brute force | Pivotaje (`tails` -> `sonic`) y escalada final | [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Tproot](../Laboratorios/DockerLabs/MuyFacil/Tproot.md) | FTP version desactualizada | [Pendiente] | [FTP.md](../Pentesting%20Notes/1_Enumeration/FTP.md) |
| [Vacaciones](../Laboratorios/DockerLabs/MuyFacil/Vacaciones.md) | Fuzzing recursivo, SSH brute force, lectura de correo local | Sudoers (Ruby) | [HTTP & HTTPS.md](../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |

### Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [Aguademayo](../Laboratorios/DockerLabs/Facil/Aguademayo.md) | Desofuscacion Brainfuck en codigo fuente Apache, fuzzing de directorios (`/images`) y SSH | Sudoers (bettercap) -> SUID (`chmod u+s /bin/bash`) -> `/bin/bash -p` | [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [ApiBase](../Laboratorios/DockerLabs/Facil/ApiBase.md) | Fuzzing de parametros y SQL Injection | Analisis de pcap (credenciales FTP de root) | [SQL Injection Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat%20Sheet.md), [FTP.md](../Pentesting%20Notes/1_Enumeration/FTP.md) |
| [Bypassme](../Laboratorios/DockerLabs/Facil/Bypassme.md) | SQL Injection (login bypass) y LFI (logs.txt) | Sudoers (socket conx) -> Script Cron (backup.sh SUID) | [SQL Injection Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat%20Sheet.md), [Services.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Services.md) |
| [Duque](../Laboratorios/DockerLabs/Facil/Duque.md) | SQL Injection, Enumeracion de facturas (Crunch) y SSH | SUID (env) | [SQL Injection Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat%20Sheet.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Elevator](../Laboratorios/DockerLabs/Facil/Elevator.md) | Subida de archivos (bypass de extension) y RCE | Sudoers en cadena (env, ash, ruby, lua, gcc, sudo) | [Path Traversal Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat%20Sheet.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Gotham](../Laboratorios/DockerLabs/Facil/Gotham.md) | Bypass de JWT (HS256 crackeo) y RCE por ping | Sudoers (find) | [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Internal](../Laboratorios/DockerLabs/Facil/Internal.md) | Fuzzing VHosts (`backup.internal.dl`), Command Injection (Bypass WAF con base64/printf/python3) | Fuerza bruta SSH con pass de backup -> SUID/Binario personalizado (`vaultctl`) | [HTTP & HTTPS.md](../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Internship](../Laboratorios/DockerLabs/Facil/Internship.md) | Virtual hosting (`gatekeeperhr.com`), SQL Injection bypass, descifrado ROT13, SSH | Escritura de script cron (`pedro` -> `valentina`) -> Esteganografía (`steghide`) -> Sudoers (`vim`) | [SQL Injection Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat%20Sheet.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Patriaquerida](../Laboratorios/DockerLabs/Facil/Patriaquerida.md) | Path Traversal / LFI y lectura de .hidden_pass | SUID (Python3) | [Path Traversal Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat%20Sheet.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [PingCTF](../Laboratorios/DockerLabs/Facil/PingCTF.md) | Command Injection en utilidad de ping (`ping.php?target=...`) y reverse shell | SUID (`/usr/bin/vim.basic` mediante Python3 exec) | [HTTP & HTTPS.md](../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [Showtime](../Laboratorios/DockerLabs/Facil/Showtime.md) | SQL Injection (SQLMap dump), RCE por panel Python, Fuerza bruta de contraseña (wordlist limpia) y SSH/su | Pivotaje (`joe` -> `luciano` via sudo posh) -> Escritura y ejecucion de script como root (`script.sh`) | [SQL Injection Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat%20Sheet.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |
| [WalkingDead](../Laboratorios/DockerLabs/Facil/WalkingDead.md) | Descubrimiento de enlace oculto en codigo fuente Apache (`/hidden/.shell.php`), RCE en parametro GET y shell inversa | SUID (Python3) | [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |

---

## Vulnyx

### Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [Swamp](../Laboratorios/Vulnyx/Facil/Swamp.md) | Transferencia de zona DNS (AXFR), Fuzzing de subdominios, Deofuscación de Javascript (Packer) | Sudoers (Inyección de comandos en script/binario personalizado) | [HTTP & HTTPS.md](../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [Permissions.md](../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md) |

### Medios

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [Express](../Laboratorios/Vulnyx/Medio/Express.md) | Virtual Hosting (`express.nyx`), Fuzzing de API, HTTP Verb Tampering (POST en `/api/admin/users`), SSRF interno con FFUF | Explotación de SSRF a SSTI (Jinja2/Flask en puerto interno 9000) con RCE directo como root | [SSRF Cheat Sheet](../Pentesting%20Notes/Web/Vulnerabilities/03-SSRF/Cheat%20Sheet.md), [FFUF.md](../Pentesting%20Notes/Web/Fuzzing/Cheat%20Sheet.md), [HTTP & HTTPS.md](../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md) |
| [Bola](../Laboratorios/Vulnyx/Medio/Bola.md) | Virtual Hosting (`bola.nyx`), Fuzzing Web (FFUF / Dirsearch), Enumeración RSYNC (`rsync-brute`), Extracción de credenciales en extensión JS (`background.js`), IDOR (MD5 hash) en panel admin (`download.php`) y SSH | Port Forwarding SSH (puerto 9000), Explotación de servicio SOAP / WSDL (Spyne) con bypass de restricción (SOAPAction vs etiqueta XML) para RCE directo como root | [HTTP & HTTPS.md](../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [SSH.md](../Pentesting%20Notes/1_Enumeration/SSH.md), [Cheat Sheet.md](../Pentesting%20Notes/Web/Fuzzing/Cheat%20Sheet.md) |
