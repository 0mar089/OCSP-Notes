
Hacemos el escaneo nmap y nos encontramos con el puerto 80 HTTP y el puerto 22 SSH abierto:

![[Pasted image 20260629235752.png]]

Hacemos un pequeño escaneo/enumeracion/analisis en la web:

![[Pasted image 20260629235837.png]]

![[Pasted image 20260630000014.png]]

Vemos que hay un apartado de bills, pero cuando entramos:

![[Pasted image 20260630000041.png]]

Nos sale un pequeño panel de inciio de sesión, y se ve que es vulnerable a *SQL Injecton*

Pero claro, si hacemos una inyección normal entramos como user (mario):

```sql
admin' or 1=1 -- -
```

![[Pasted image 20260630000141.png]]

Hay que intentar poder entrar como administradores, asi que probamos varios payloads:

```
admin' and 1=1-- -
```

He pensado que seguramente, el codigo asigne un rol (tambien en la sesion PHP) dependiendo del usuario que eres. Entonces si yo pongo **admin' or 1=1-- -**, en la query, el resultado será el 1=1 y no el user de admin solo, entonces me devuelve todos los usuarios (ya que el 1=1 siempre es true).  SI pongo el *AND* entonces la condicion tiene que ser true tanto el del usuario admin como el 1=1. 

Por lo tanto me pillará el user admin. 

Entonces me aparece otro panel:
![[Pasted image 20260630000956.png]]

Aqui hay un posible IDOR, ya que la barra de busqueda añade a la url un id tal que asó:

```
http://172.17.0.2/bills/panel.php?id=fd
```

Lo que pasa es que las facturas parecen tener un patron de tres letras y tres numeros:

```
id=xya456
```

![[Pasted image 20260630001110.png]]

Asi que con un comando, haremos una wordlist en busca de otras facturas a ver si podemos encontrar información:

```bash
crunch 6 6 -t xy@%%% -o diccionario_corto.txt
```

Esto genera un diccionario llamado **diccionario_corto.txt** que tiene 6 caracteres minimos y 6 maximo, asi que solo de 6 caracteres. Y con una plantilla de tipo, los 2 primeros caracteres serán xy, el proximo será una letra minuscula de la a-z (comodín @) y comodines de numeros (del 0 al 9) con el simbolo %. 

Entonces el ataque quedaria tal que así:
![[Pasted image 20260630001642.png]]

Y se ve que el id=xyc724 cambia de entre todos. Y si, se ven credenciales de usuario en esa pagina:

![[Pasted image 20260630001725.png]]

Probamos a hacer ssh, y efectivamente nos deja:
![[Pasted image 20260630001811.png]]

Ahora probamos a escalar privilegios:

Vemos que viendo los permisos SUID, podemos encontrar una vulnerabilidad con el archivo *env*:
![[Pasted image 20260630193443.png]]

Asi que podmeos ejecutar un comando para escalar privilegios de *duque* a *root*:

```bash
/usr/bin/env /bin/bash -p
```




