## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo inicial con nmap para enumerar los servicios activos de la máquina objetivo:

![[Pasted image 20260214175919.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Puerto abierto para control remoto seguro. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Reconocimiento de Tecnologías (WhatWeb)
Evaluamos las tecnologías utilizadas por el sitio web mediante whatweb:

![[Pasted image 20260214180028.png]]

No se muestran indicios de CMS o frameworks vulnerables. 

### Inspección Visual e Inicial
Accedemos al sitio web a través del navegador:

![[Pasted image 20260214180110.png]]

* **Hallazgo:** La página web se encuentra totalmente en blanco. Lo único rescatable son comentarios y referencias en el código fuente. Decidimos realizar fuzzing de directorios.

### Fuzzing de Directorios Recursivo
Iniciamos la búsqueda de directorios ocultos. Tras identificar ciertas rutas iniciales, realizamos un fuzzing recursivo que nos revela directorios anidados:

![[Pasted image 20260214182642.png]]

Fuzzeamos la ruta /javascript encontrada:

![[Pasted image 20260214182710.png]]

Y repetimos la búsqueda de recursos hasta obtener códigos de respuesta exitosos (200 OK):

![[Pasted image 20260214182730.png]]

* **Resultado:** La ruta final encontrada contiene únicamente la librería jquery, por lo que descartamos vectores de ataque directos por esta vía web.

---

## Fase de Explotación / Intrusión

### Fuerza Bruta SSH (Acceso Inicial)
A partir de la enumeración anterior, identificamos posibles usuarios en el sistema. Procedemos a realizar un ataque de fuerza bruta por SSH contra estas cuentas:

![[Pasted image 20260214183345.png]]

* **Credencial encontrada:** Logramos obtener una contraseña válida para uno de los usuarios analizados.
* **Acceso:** Iniciamos sesión vía SSH en el servidor:

![[Pasted image 20260214183820.png]]

### Pivotaje de Usuario (juan)
Durante la fase de enumeración interna tras el acceso inicial, revisamos el correo del sistema local. Encontramos un correo enviado por Juan que contiene una contraseña en texto claro:

* **Acción:** Usamos la contraseña obtenida para conectarnos nuevamente vía SSH, esta vez como el usuario juan:

![[Pasted image 20260214183941.png]]

---

## Escalada de Privilegios

### Enumeración Interna
Una vez dentro del sistema con la cuenta de juan, buscamos binarios o permisos especiales que nos permitan elevar privilegios:

```bash
sudo -l
```

![[Pasted image 20260214184020.png]]

* **Vulnerabilidad de Configuración:** El usuario juan tiene permisos para ejecutar el intérprete de programación /usr/bin/ruby con máximos privilegios (root) sin ingresar contraseña (NOPASSWD). Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].

### Explotación de Ruby (Sudoers)
Utilizamos el intérprete de ruby para ejecutar código del sistema que nos retorne una shell interactiva como superusuario:

Ejecutamos:
```bash
sudo ruby -e 'exec "/bin/bash"'
```

![[Pasted image 20260214184109.png]]

¡Conseguimos con éxito una shell interactiva como el usuario root!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/MuyFacil/Hedgehog\|Hedgehog]] (Pivotaje de usuarios), [[Laboratorios/DockerLabs/Facil/Elevator\|Elevator]] (Comparte pivotaje por ruby)