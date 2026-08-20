## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![Pasted image 20260816232730.png](../../../assets/Pasted%20image%2020260816232730.png)

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Servicio OpenSSH para acceso remoto por terminal. Ver teoría en [SSH.md](../../../Pentesting%20Notes/1_Enumeration/SSH.md).
* **Puerto 80/TCP (HTTP):** Servidor web HTTP en ejecución. Ver teoría en [HTTP & HTTPS.md](../../../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md).

---

## Enumeración Web y de API

### Inspección del Servidor Web (WhatWeb y Virtual Hosting)
Accedemos a la dirección IP a través del navegador por el puerto 80, encontrándonos con la página por defecto de Apache:

![Pasted image 20260816232851.png](../../../assets/Pasted%20image%2020260816232851.png)

El análisis inicial con `whatweb` no revela tecnologías adicionales de interés. Para comprobar si el servidor utiliza Virtual Hosting, añadimos el dominio `express.nyx` en nuestro archivo `/etc/hosts`:

```text
192.168.1.X express.nyx
```

Al recargar el navegador utilizando el dominio `express.nyx`, el servidor sirve una nueva página web corporativa:

![Pasted image 20260816235801.png](../../../assets/Pasted%20image%2020260816235801.png)

### Inspección del Código Fuente y Endpoints de la API
Revisamos el código fuente de la página web y descubrimos la carga de dos scripts JavaScript:

![Pasted image 20260816235902.png](../../../assets/Pasted%20image%2020260816235902.png)

Al inspeccionar el contenido del archivo `api.js`:

![Pasted image 20260817000027.png](../../../assets/Pasted%20image%2020260817000027.png)

Identificamos la definición de dos endpoints principales pertenecientes a una API administrativa:
1. `/api/admin/availability`
2. `/api/admin/users?key={key}`

---

## Fase de Explotación / Intrusión

### HTTP Verb Tampering y Fuga de Credenciales/Tokens
Procedemos a evaluar el comportamiento de los endpoints encontrados.

Al enviar una petición GET al endpoint `/api/admin/availability`, el servidor deniega el acceso indicando que se requiere un token de autenticación:

![Pasted image 20260817000957.png](../../../assets/Pasted%20image%2020260817000957.png)

A continuación, probamos el endpoint `/api/admin/users`:

![Pasted image 20260817001633.png](../../../assets/Pasted%20image%2020260817001633.png)

Al realizar **HTTP Verb Tampering** (cambiando el método de la petición HTTP de `GET` a `POST`), logramos evadir la validación de la clave/token:

![Pasted image 20260817001657.png](../../../assets/Pasted%20image%2020260817001657.png)

El servidor nos devuelve una estructura JSON con todos los usuarios registrados y sus respectivos tokens de acceso. Identificamos al usuario con rol de administrador (`admin`):

```json
{
  "id": 18,
  "roles": [
    "admin"
  ],
  "token": "4493-3179-0912-0597",
  "username": "JESSS"
}
```

### Explotación de Server-Side Request Forgery (SSRF)
Con el token de administrador obtenido (`4493-3179-0912-0597`), interactuamos con el endpoint `/api/admin/availability` enviando un payload en formato JSON:

![Pasted image 20260817002116.png](../../../assets/Pasted%20image%2020260817002116.png)

* **Análisis de Vulnerabilidad:** El parámetro `url` dentro del cuerpo de la petición es interpretado y consultado directamente por el servidor backend, lo que nos permite explotar una vulnerabilidad de **Server-Side Request Forgery (SSRF)**. Ver teoría en [SSRF Cheat Sheet](../../../Pentesting%20Notes/Web/Vulnerabilities/03-SSRF/Cheat%20Sheet.md).

Probamos inicialmente si el parámetro permite lectura de archivos locales mediante el esquema `file://`:

![Pasted image 20260818151937.png](../../../assets/Pasted%20image%2020260818151937.png)

Al no obtener respuesta con `file://`, procedemos a utilizar el vector SSRF para realizar un escaneo de puertos internos hacia la interfaz de loopback (`127.0.0.1`).

### Fuzzing de Puertos Internos con FFUF
Utilizamos `ffuf` para automatizar las peticiones POST contra el endpoint vulnerable, variando el puerto en la URL interna:

```bash
ffuf -u "http://express.nyx/api/admin/availability" -X POST -H "Content-Type: application/json" -d '{"id": 1, "url": "http://127.0.0.1:FUZZ", "token": "4493-3179-0912-0597"}' -w ports.txt -ic -c -fw 36
```

Ver guía y sintaxis de filtrado en [FFUF.md](../../../Pentesting%20Notes/Web/Fuzzing/Cheat%20Sheet.md).

![Pasted image 20260818160930.png](../../../assets/Pasted%20image%2020260818160930.png)

* **Hallazgo:** El escaneo revela dos puertos internos abiertos: el puerto **5000** y el puerto **9000**.

### Inspección de Servicios Internos (Puertos 5000 y 9000)
Consultamos ambos puertos a través de la vulnerabilidad SSRF:

**Puerto 9000:**
![Pasted image 20260818161058.png](../../../assets/Pasted%20image%2020260818161058.png)

**Puerto 5000:**
![Pasted image 20260818161115.png](../../../assets/Pasted%20image%2020260818161115.png)

El servicio del puerto 9000 muestra una estructura web que acepta peticiones GET en la ruta `/username` con el parámetro `name`:

```json
"url": "http://127.0.0.1:9000/username?name=admin"
```

Al realizar la petición obtenemos la siguiente respuesta:

![Pasted image 20260818161420.png](../../../assets/Pasted%20image%2020260818161420.png)

---

## Escalada de Privilegios / Ejecución Remota de Código (RCE)

### Detección de Server-Side Template Injection (SSTI)
Observamos que el valor proporcionado en el parámetro `name` se refleja directamente en la plantilla HTML generada por el servidor sin sanitización:

![Pasted image 20260818161452.png](../../../assets/Pasted%20image%2020260818161452.png)

Probamos un payload de detección básica de **SSTI** inyectando una operación matemática:

```text
http://127.0.0.1:9000/username?name={{7*7}}
```

![Pasted image 20260818161533.png](../../../assets/Pasted%20image%2020260818161533.png)

* **Resultado:** El motor de plantillas evalúa la expresión aritmética devolviendo `49`, confirmando la presencia de una inyección de plantillas del lado del servidor (SSTI).

### Identificación del Motor y RCE como Root
Para identificar el motor de plantillas y explorar el entorno de ejecución, inyectamos el objeto `{{config}}`:

![Pasted image 20260818161900.png](../../../assets/Pasted%20image%2020260818161900.png)

La respuesta confirma un entorno Python con Flask / Jinja2. Construimos un payload en Jinja2 para acceder a los módulos globales de Python (`__globals__`), importar el módulo `os` y ejecutar comandos del sistema mediante `os.popen()`:

```jinja2
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Enviamos el payload a través de la petición SSRF:

![Pasted image 20260818162511.png](../../../assets/Pasted%20image%2020260818162511.png)

* **Impacto Crítico:** La salida del comando (`uid=0(root) gid=0(root) groups=0(root)`) confirma que el servicio interno en el puerto 9000 corre directamente bajo la identidad del superusuario **root**. Al lograr RCE en este contexto, alcanzamos el máximo nivel de privilegios directamente.

### Lectura de la Flag de Root
Ejecutamos la lectura del archivo de la flag ubicado en `/root/root.txt`:

![Pasted image 20260818162707.png](../../../assets/Pasted%20image%2020260818162707.png)

¡Control total obtenido sobre la máquina objetivo!

---

## Relaciones y Conceptos
* **Teoría:** [SSRF Cheat Sheet](../../../Pentesting%20Notes/Web/Vulnerabilities/03-SSRF/Cheat%20Sheet.md), [FFUF.md](../../../Pentesting%20Notes/Web/Fuzzing/Cheat%20Sheet.md), [HTTP & HTTPS.md](../../../Pentesting%20Notes/1_Enumeration/HTTP%20%26%20HTTPS.md), [SSH.md](../../../Pentesting%20Notes/1_Enumeration/SSH.md), [Linux Privilege Escalation - Permissions.md](../../../Pentesting%20Notes/3_Post-Explotation/Linux%20Privilage%20Escalation/Permissions.md)
* **Laboratorios Relacionados:** [Swamp](../../../Laboratorios/Vulnyx/Facil/Swamp.md) (Comparte resolución por VHosts y análisis de archivos JavaScript), [Gotham](../../../Laboratorios/DockerLabs/Facil/Gotham.md) (Comparte interacción con paneles y explotación de endpoints vulnerables)
