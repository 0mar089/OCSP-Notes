# Indice General de Laboratorios

## DockerLabs

### Muy Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [[Laboratorios/DockerLabs/MuyFacil/Obsession\|Obsession]] | FTP Anonimo, Fuzzing Web (`/backup`) y SSH brute force | Sudoers (Vim) | [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Trust\|Trust]] | Fuzzing Web (`secret.php`) y SSH brute force | Sudoers (Vim) | [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Hedgehog\|Hedgehog]] | Inversion de diccionario (Rockyou) y SSH brute force | Pivotaje (`tails` -> `sonic`) y escalada final | [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Tproot\|Tproot]] | FTP version desactualizada | [Pendiente] | [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Vacaciones\|Vacaciones]] | Fuzzing recursivo, SSH brute force, lectura de correo local | Sudoers (Ruby) | [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |

### Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [[Laboratorios/DockerLabs/Facil/Aguademayo\|Aguademayo]] | Desofuscacion Brainfuck en codigo fuente Apache, fuzzing de directorios (`/images`) y SSH | Sudoers (bettercap) -> SUID (`chmod u+s /bin/bash`) -> `/bin/bash -p` | [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/ApiBase\|ApiBase]] | Fuzzing de parametros y SQL Injection | Analisis de pcap (credenciales FTP de root) | [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]] |
| [[Laboratorios/DockerLabs/Facil/Bypassme\|Bypassme]] | SQL Injection (login bypass) y LFI (logs.txt) | Sudoers (socket conx) -> Script Cron (backup.sh SUID) | [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Services\|Services.md]] |
| [[Laboratorios/DockerLabs/Facil/Duque\|Duque]] | SQL Injection, Enumeracion de facturas (Crunch) y SSH | SUID (env) | [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Elevator\|Elevator]] | Subida de archivos (bypass de extension) y RCE | Sudoers en cadena (env, ash, ruby, lua, gcc, sudo) | [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Gotham\|Gotham]] | Bypass de JWT (HS256 crackeo) y RCE por ping | Sudoers (find) | [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Internal\|Internal]] | Fuzzing VHosts (`backup.internal.dl`), Command Injection (Bypass WAF con base64/printf/python3) | Fuerza bruta SSH con pass de backup -> SUID/Binario personalizado (`vaultctl`) | [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Internship\|Internship]] | Virtual hosting (`gatekeeperhr.com`), SQL Injection bypass, descifrado ROT13, SSH | Escritura de script cron (`pedro` -> `valentina`) -> Esteganografía (`steghide`) -> Sudoers (`vim`) | [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Patriaquerida\|Patriaquerida]] | Path Traversal / LFI y lectura de .hidden_pass | SUID (Python3) | [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Showtime\|Showtime]] | SQL Injection (SQLMap dump), RCE por panel Python, Fuerza bruta de contraseña (wordlist limpia) y SSH/su | Pivotaje (`joe` -> `luciano` via sudo posh) -> Escritura y ejecucion de script como root (`script.sh`) | [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/WalkingDead\|WalkingDead]] | Descubrimiento de enlace oculto en codigo fuente Apache (`/hidden/.shell.php`), RCE en parametro GET y shell inversa | SUID (Python3) | [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |

---

## Vulnyx

### Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [[Laboratorios/Vulnyx/Facil/Swamp\|Swamp]] | Transferencia de zona DNS (AXFR), Fuzzing de subdominios, Deofuscación de Javascript (Packer) | Sudoers (Inyección de comandos en script/binario personalizado) | [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |

