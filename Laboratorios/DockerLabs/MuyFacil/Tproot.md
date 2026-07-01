## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo inicial de puertos en la máquina objetivo para identificar servicios activos:

![[Pasted image 20260618225121.png]]

**Servicios identificados:**
* **Puerto 21/TCP (FTP):** Servicio de transferencia de archivos activo. Ver teoría en [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]].
* **Puerto 80/TCP (HTTP):** Servidor web activo. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración de Servicios y Vulnerabilidades

### Análisis del Servicio FTP
Inspeccionamos la versión del servicio FTP que está corriendo en el servidor. Al ser una versión antigua, procedemos a verificar si cuenta con vulnerabilidades públicas explotables:

![[Pasted image 20260618225158.png]]

* **Resultado:** [Explicar aquí el exploit o vulnerabilidad identificada para la versión de FTP y cómo se procedió a explotarlo].

---

## Fase de Explotación / Intrusión

* [Documentar el método de ejecución del exploit, las opciones configuradas y cómo se obtuvo el acceso inicial a la máquina].

---

## Escalada de Privilegios

* [Documentar los comandos de enumeración ejecutados en el sistema víctima y los pasos realizados para obtener privilegios de root].

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/Facil/ApiBase\|ApiBase]] (Comparte análisis relacionado con servicios FTP)