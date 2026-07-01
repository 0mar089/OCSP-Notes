## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Iniciamos con un escaneo de puertos sobre la máquina objetivo para listar los servicios expuestos:

![[Pasted image 20260218193032.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Servicio de acceso seguro SSH abierto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web expuesto en el puerto estándar. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Reconocimiento de Tecnologías e Inspección
Realizamos un escaneo web básico con whatweb para identificar tecnologías:

![[Pasted image 20260218193109.png]]

No se aprecian tecnologías complejas ni gestores de contenido. Hacemos una petición directa usando curl o el navegador para revisar el código fuente y contenido visible:

![[Pasted image 20260218193202.png]]

* **Resultado:** La página web únicamente muestra el texto "tails". Esto nos sugiere un posible nombre de usuario para futuras fases.

### Fuzzing de Directorios
Procedemos a realizar fuzzing de directorios para verificar si existen rutas ocultas:

![[Pasted image 20260218193241.png]]

No se obtienen resultados fructíferos ni recursos adicionales a través del fuzzing web.

---

## Fase de Explotación / Intrusión

### Preparación del Diccionario y Fuerza Bruta SSH
Considerando el indicio del usuario tails, nos disponemos a realizar un ataque de fuerza bruta SSH. Para optimizar el ataque en este entorno, decidimos invertir el orden de la wordlist rockyou.txt y remover los espacios en blanco:

Invertir el diccionario:
```bash
tac /usr/share/wordlists/rockyou.txt > rockyou_invertido.txt
```

Eliminar posibles espacios en blanco:
```bash
sed -i 's/ //g' rockyou_invertido.txt
```

Lanzamos el ataque de fuerza bruta contra el servicio SSH utilizando el diccionario modificado:

![[Pasted image 20260218193506.png]]

* **Credencial encontrada:** tails:password (identificada con éxito).
* **Acceso:** Iniciamos sesión en el sistema vía SSH.

---

## Escalada de Privilegios

### Pivotaje de Usuario (tails -> sonic)
Una vez dentro del sistema con el usuario tails, enumeramos los permisos de ejecución de sudo para ver qué binarios o comandos podemos ejecutar como otros usuarios:

```bash
sudo -l
```

![[Pasted image 20260218194223.png]]

* **Hallazgo:** El usuario tails puede ejecutar cualquier comando como el usuario sonic sin ingresar contraseña (NOPASSWD). Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].
* **Acción:** Pivotamos al usuario sonic ejecutando una shell en su nombre:

```bash
sudo -u sonic /bin/bash
```

![[Pasted image 20260218194307.png]]

### Escalada Final a Root
Una vez posicionados como el usuario sonic, buscamos la forma de escalar privilegios para convertirnos en root:

![[Pasted image 20260218194326.png]]

[Detalle de la obtención de la shell de root finalizada con éxito].

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]], [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/MuyFacil/Vacaciones\|Vacaciones]] (Usa pivotaje de usuarios)
