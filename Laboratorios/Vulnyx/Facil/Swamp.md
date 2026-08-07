## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo de puertos inicial con nmap para identificar los servicios activos en la máquina objetivo:

![[Pasted image 20260805224509.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Servicio OpenSSH abierto para acceso remoto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 53/TCP (DNS):** Servidor DNS en ejecución.
* **Puerto 80/TCP (HTTP):** Servidor web Apache. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

* **Resolución de Nombres:** Identificamos que el sitio web intenta redirigir al dominio `swamp.nyx`. Procedemos a añadirlo a nuestro archivo `/etc/hosts`:
```text
192.168.1.145  swamp.nyx
```

---

## Enumeración de Servicios y Web

### Transferencia de Zona DNS (AXFR)
Dado que el puerto 53 de DNS está abierto, intentamos realizar una transferencia de zona (AXFR). Esta vulnerabilidad ocurre cuando un servidor DNS responde a solicitudes de transferencia de zona de cualquier host, permitiendo descargar la base de datos completa de nombres de dominio y subdominios del sistema.

Ejecutamos dig para solicitar la transferencia de zona:

```bash
dig axfr @192.168.1.145 swamp.nyx
```

![[Pasted image 20260805225355.png]]

* **Subdominios Descubiertos:** La transferencia de zona expone múltiples subdominios activos. Los registramos en nuestro `/etc/hosts`:
```text
192.168.1.145 d0nkey.swamp.nyx
192.168.1.145 dr4gon.swamp.nyx
192.168.1.145 duloc.swamp.nyx
192.168.1.145 f1ona.swamp.nyx
192.168.1.145 farfaraway.swamp.nyx
192.168.1.145 shr3k.swamp.nyx
```

### Inspección de Subdominios y Fuzzing
Inspeccionamos cada uno de los subdominios a través del navegador. En `d0nkey.swamp.nyx` encontramos una referencia a un archivo `video.mp4` en el código fuente.

Para descartar recursos ocultos en los demás subdominios, creamos y ejecutamos un script en bash para realizar fuzzing automatizado mediante ffuf:

```bash
#!/bin/bash
WORDLIST="/usr/share/wordlist/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-lowercase-2.3-big.txt"
FILTER="404"
OUTPUT_DIR="resultados"
THREADS="30"

mkdir -p "$OUTPUT_DIR"

while read -r subdominio <&3; do
    ffuf -w "$WORDLIST" \
         -u "http://$subdominio/FUZZ" \
         -c \
         -fc "$FILTER" \
         -ic \
         -t "$THREADS" \
         -o "$OUTPUT_DIR/$subdominio.json" \
         -of json
done 3< subdominios.txt
```

El fuzzing no reporta directorios adicionales.

---

## Fase de Explotación / Intrusión

### Análisis de Código y Deofuscación de Javascript
Al analizar detalladamente el código fuente de `farfaraway.swamp.nyx`, identificamos un bloque de código JavaScript ofuscado:

![[Pasted image 20260808005938.png]]

* **Deofuscación:** El script utiliza una ofuscación de tipo Packer. Utilizamos herramientas de deofuscación (como UNPacker o herramientas online de formateo) para revelar su contenido original.
* **Credenciales Encontradas:** El script expone en texto plano las credenciales del usuario:
```text
shrek:putopesaoelasno
```

### Acceso SSH
Realizamos la conexión mediante SSH utilizando las credenciales obtenidas:

```bash
ssh shrek@192.168.1.145
```

![[Pasted image 20260808010028.png]]

* **Acceso:** Obtenemos una shell inicial en el sistema bajo la identidad del usuario shrek.

---

## Escalada de Privilegios

### Enumeración Interna y Sudoers
Como usuario shrek, comprobamos los permisos de sudo asignados en el sistema:

```bash
sudo -l
```

![[Pasted image 20260808010159.png]]

* **Vulnerabilidad de Configuración:** El usuario shrek está autorizado a ejecutar un comando/script personalizado con privilegios de root sin proporcionar contraseña.

### Command Injection en Binario Personalizado
Al interactuar con el binario autorizado, observamos que permite pasar argumentos. Al ingresar comandos del sistema o caracteres especiales en los campos esperados, se ejecuta el comando directamente en el contexto del sistema.

Inyectamos un comando directamente en el parámetro/cabecera esperado:

![[Pasted image 20260808010308.png]]

Para elevar privilegios, inyectamos la ejecución de una terminal bash:

```bash
/bin/bash
```

![[Pasted image 20260808010345.png]]

* **Resultado:** Obtenemos acceso completo como root con privilegios totales en la máquina.

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/Facil/Internal\|Internal]] (Comparte resolución de hosts y descubrimiento de subdominios)