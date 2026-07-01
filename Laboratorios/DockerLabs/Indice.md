# Indice de Laboratorios - DockerLabs

## Muy Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [[Laboratorios/DockerLabs/MuyFacil/Obsession\|Obsession]] | FTP Anonimo, Fuzzing Web (`/backup`) y SSH brute force | Sudoers (Vim) | [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Trust\|Trust]] | Fuzzing Web (`secret.php`) y SSH brute force | Sudoers (Vim) | [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Hedgehog\|Hedgehog]] | Inversion de diccionario (Rockyou) y SSH brute force | Pivotaje (`tails` -> `sonic`) y escalada final | [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Tproot\|Tproot]] | FTP version desactualizada | [Pendiente] | [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]] |
| [[Laboratorios/DockerLabs/MuyFacil/Vacaciones\|Vacaciones]] | Fuzzing recursivo, SSH brute force, lectura de correo local | Sudoers (Ruby) | [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |

## Faciles

| Laboratorio | Vector de Intrusion | Escalada de Privilegios | Conceptos Relacionados |
| --- | --- | --- | --- |
| [[Laboratorios/DockerLabs/Facil/ApiBase\|ApiBase]] | Fuzzing de parametros y SQL Injection | Analisis de pcap (credenciales FTP de root) | [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]] |
| [[Laboratorios/DockerLabs/Facil/Duque\|Duque]] | SQL Injection, Enumeracion de facturas (Crunch) y SSH | SUID (env) | [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Elevator\|Elevator]] | Subida de archivos (bypass de extension) y RCE | Sudoers en cadena (env, ash, ruby, lua, gcc, sudo) | [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Gotham\|Gotham]] | Bypass de JWT (HS256 crackeo) y RCE por ping | Sudoers (find) | [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
| [[Laboratorios/DockerLabs/Facil/Patriaquerida\|Patriaquerida]] | Path Traversal / LFI y lectura de .hidden_pass | SUID (Python3) | [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]] |
