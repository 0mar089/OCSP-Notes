## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial para mapear los servicios expuestos:

![[Pasted image 20260222170431.png]]

**Servicios identificados:**
* **Puerto 5000/TCP (HTTP/API):** Servidor web que expone una interfaz API. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración de la API Web

### Inspección de Endpoints
Accedemos a la API web en el puerto 5000:

![[Pasted image 20260222170501.png]]

El servidor nos informa sobre dos rutas principales: /add y /users.
* Al intentar acceder a /add, el servidor responde con un error Method Not Allowed (método HTTP no permitido).
* Al acceder a /users, la ruta responde exitosamente, pero indica la falta de parámetros:

![[Pasted image 20260222170549.png]]

### Fuzzing de Parámetros
Para descubrir qué parámetro está esperando recibir el endpoint /users, realizamos un fuzzing de parámetros:

![[Pasted image 20260222170639.png]]

Para refinar la búsqueda, excluimos las respuestas con una longitud de 35 caracteres (--exclude-length 35):

![[Pasted image 20260222170708.png]]

* **Hallazgo:** Descubrimos el parámetro válido necesario para realizar consultas al endpoint de usuarios.

---

## Fase de Explotación / Intrusión

### Identificación y Explotación de SQLi
Usando el parámetro descubierto, enviamos nombres de prueba. Al ingresar el nombre d'anne, la aplicación web devuelve un error de sintaxis de base de datos SQL:

![[Pasted image 20260222171122.png]]

Confirmamos la presencia de un error SQL:

![[Pasted image 20260222171157.png]]

Aprovechamos este error para ejecutar una inyección SQL (SQLi) clásica para saltarse o extraer información de la base de datos:

![[Pasted image 20260222172119.png]]

* **Resultado:** Logramos enumerar y extraer nombres de usuarios válidos del sistema. Ver guía en [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]]. Con este listado de usuarios, realizamos un intento de conexión por SSH:

![[Pasted image 20260222173100.png]]

Establecemos con éxito una sesión en el servidor.

---

## Escalada de Privilegios

### Enumeración Interna y Descarga de PCAP
Exploramos los archivos del sistema y localizamos el código fuente de la API programada en Python. Adicionalmente, detectamos un archivo de captura de tráfico de red (Wireshark / PCAP).

Para analizar el archivo .pcap en nuestra máquina local, levantamos un servidor HTTP rápido con Python en la máquina víctima para transferirlo:

```bash
python3 -m http.server 8000
```

Descargamos el archivo y lo abrimos con Wireshark:

![[Pasted image 20260222173217.png]]

* **Hallazgo Crítico:** Analizando las tramas de red capturadas correspondientes al servicio FTP, logramos extraer las credenciales en texto plano enviadas durante el proceso de autenticación. Ver teoría en [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]].
* **Credenciales de Root encontradas:** root:balulero

### Escalada Final
Utilizamos la contraseña obtenida para autenticarnos directamente como el usuario root en el sistema.

¡Acceso completo a la máquina finalizado!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/1_Enumeration/FTP\|FTP.md]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/Facil/Duque\|Duque]] (Comparte uso de SQLi)
