## Fase de Reconocimiento y Descubrimiento (Enumeración)

### Escaneo de Puertos (Nmap)
Realizamos el escaneo inicial con nmap e identificamos el puerto 80 HTTP y el puerto 22 SSH abierto:

![[Pasted image 20260629235752.png]]

**Servicios identificados:**
* **Puerto 22/TCP (SSH):** Puerto abierto para control remoto. Ver teoría en [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]].
* **Puerto 80/TCP (HTTP):** Servidor web expuesto. Ver teoría en [[Pentesting Notes/1_Enumeration/HTTP & HTTPS\|HTTP & HTTPS.md]].

---

## Enumeración Web

### Análisis Web e Inicio de Sesión
Procedemos a realizar el análisis del servidor web:

![[Pasted image 20260629235837.png]]

![[Pasted image 20260630000014.png]]

Al navegar por la web, identificamos un apartado de facturas ("bills"). Al acceder, nos muestra un panel de inicio de sesión:

![[Pasted image 20260630000041.png]]

El panel parece vulnerable a inyección SQL (SQLi).

---

## Fase de Explotación / Intrusión

### Explotación de SQL Injection y Bypass de Login
Al realizar una inyección SQL clásica para saltarse la autenticación, ingresamos inicialmente como el usuario de menor rango mario:

```sql
admin' or 1=1 -- -
```

![[Pasted image 20260630000141.png]]

Para acceder como administrador y ver más opciones, refinamos el payload. El objetivo es que la base de datos devuelva la fila de admin específicamente:

```sql
admin' and 1=1-- -
```

Al utilizar este payload, logramos acceder al panel administrativo:

![[Pasted image 20260630000956.png]]

Ver guía teórica sobre inyecciones SQL en [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]].

### Explotación de IDOR / Enumeración de Facturas
El panel administrativo expone una búsqueda que añade un parámetro ID a la URL:

```text
http://172.17.0.2/bills/panel.php?id=fd
```

Las facturas válidas parecen seguir una nomenclatura específica de tres letras seguidas de tres números (ej. xya456):

![[Pasted image 20260630001110.png]]

Para enumerar todas las facturas en busca de datos sensibles, generamos una wordlist personalizada con crunch utilizando un patrón específico (dos letras fijas "xy", seguidas de un carácter alfabético comodín y tres caracteres numéricos comodines):

```bash
crunch 6 6 -t xy@%%% -o diccionario_corto.txt
```

Utilizamos este diccionario en la búsqueda de IDs válidos:

![[Pasted image 20260630001642.png]]

* **Hallazgo:** Identificamos la factura con el ID xyc724. Al acceder a ella, se muestran credenciales de usuario del sistema en texto claro:

![[Pasted image 20260630001725.png]]

* **Acceso:** Probamos las credenciales obtenidas contra el servicio SSH y logramos ingresar con éxito:

![[Pasted image 20260630001811.png]]

---

## Escalada de Privilegios

### Enumeración Interna y SUID
Listamos los binarios del sistema que tienen configurados permisos SUID de ejecución:

![[Pasted image 20260630193443.png]]

* **Vulnerabilidad de Configuración:** El comando env (/usr/bin/env) tiene el bit SUID activo. Ver teoría en [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Permissions.md]].

### Explotación de SUID (env)
Aprovechamos el bit SUID del binario env para forzar la ejecución de una shell bash con privilegios elevados:

```bash
/usr/bin/env /bin/bash -p
```

¡Obtenemos acceso como el usuario root!

---

## Relaciones y Conceptos
* **Teoría:** [[Pentesting Notes/Web/Vulnerabilities/01-SQL_Injection/Cheat Sheet\|SQL Injection Cheat Sheet]], [[Pentesting Notes/3_Post-Explotation/Linux Privilage Escalation/Permissions\|Linux Privilege Escalation - Permissions.md]], [[Pentesting Notes/1_Enumeration/SSH\|SSH.md]]
* **Laboratorios Relacionados:** [[Laboratorios/DockerLabs/Facil/ApiBase\|ApiBase]] (Comparte uso de SQLi), [[Laboratorios/DockerLabs/Facil/Elevator\|Elevator]] (Comparte escalada por env)
