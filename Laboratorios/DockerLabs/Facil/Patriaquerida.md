## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos:

![[Pasted image 20260630232140.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Puerto abierto para control remoto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Inspección de Tecnologías y Servidor (WhatWeb)
Evaluamos el sitio antes de acceder mediante whatweb:

![[Pasted image 20260630232240.png]]

El servidor nos muestra la página por defecto de Apache:

![[Pasted image 20260630232308.png]]

### Fuzzing de Directorios (ffuf)
Realizamos fuzzing de archivos y directorios con ffuf, identificando el archivo `index.php`:

![[Pasted image 20260630232934.png]]

![[Pasted image 20260630232951.png]]

Al inspeccionar esta página web, encontramos una indicación de la existencia de un archivo oculto que supuestamente contiene una contraseña:

![[Pasted image 20260630233000.png]]

### Fuzzing de Parámetros y Path Traversal (LFI)
Dado que no hay otras interacciones obvias en la web, realizamos fuzzing de parámetros GET para identificar variables vulnerables. Encontramos que el parámetro `page` es aceptado por la URL:

```text
http://172.17.0.2/index.php?page=xxxxxx
```

![[Pasted image 20260630235046.png]]

Intentamos acceder a archivos locales del sistema para verificar la vulnerabilidad de Path Traversal / Local File Inclusion (LFI):

![[Pasted image 20260630235021.png]]

* **Hallazgo:** Confirmamos la vulnerabilidad de LFI. Al leer `/etc/passwd`, identificamos dos posibles usuarios en el sistema: `pinguino` y `mario`. Ver teoría sobre inyecciones de rutas en [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]].

---

## Fase de Explotación / Intrusión

### Obtención de Credenciales de Acceso (LFI)
Aunque realizamos intentos de fuerza bruta sobre el servicio SSH para obtener accesos, no resultaron exitosos. Procedemos a explotar el LFI para leer el archivo oculto `.hidden_pass` que se nos sugirió anteriormente:

![[Pasted image 20260630235159.png]]

* **Credenciales encontradas:** `pinguino:balu`
* **Acceso Inicial:** Iniciamos sesión vía SSH con el usuario `pinguino`. Al acceder, localizamos una nota con información interna:

![[Pasted image 20260630235302.png]]

### Pivotaje de Usuario (`pinguino` a `mario`)
Utilizando la información de la nota interna, establecemos una nueva conexión SSH o pivotamos al usuario `mario`:

![[Pasted image 20260630235418.png]]

---

## Escalada de Privilegios

### Enumeración Interna
Como el usuario `mario`, verificamos permisos y archivos web locales pero no encontramos nada útil. Decidimos realizar una enumeración de binarios con permisos SUID:

![[Pasted image 20260630235625.png]]

* **Vulnerabilidad de Configuración:** El intérprete de Python `/usr/bin/python3.8` tiene asignado el bit SUID. Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].

### Explotación de SUID (Python3)
Consultando las técnicas de elevación de privilegios en GTFOBins, ejecutamos el one-liner correspondiente para ejecutar una shell sh preservando el UID del superusuario (root):

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.execl("/bin/sh", "sh")'
```

¡Obtenemos una shell con privilegios de root con acceso completo al sistema!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]]
* **Laboratorios Relacionados:** [Ninguno con técnicas similares documentadas actualmente]
