Hacemos el escaneo nmap y obtenemos:
![[Pasted image 20260222170431.png]]

SI accedemos a la pagina web por el puerto 5000 vemos que es una API:
![[Pasted image 20260222170501.png]]

Nos dice que podemos usar tanto /add como /users.  SI probamos con /add nos dirá *Method Not Allowed*, supongo que es por falta de permisos. 

Pero si ponemos /users:

![[Pasted image 20260222170549.png]]

Eso significa que el endpoint es válido, podemos usarlo pero no estamos enviando los parametros. Por lo tanto podemos hacer un fuzzing de parametros disponibles:

![[Pasted image 20260222170639.png]]

Nos da lo mismo, asi que excluimos el length *35*:

![[Pasted image 20260222170708.png]]

Hemos enocntrado el parametro que podemos usar, probablemente para enumerar usuarios. 

Asi que volvemos a hacer fuzzing y encontramos:

![[Pasted image 20260222171122.png]]

Si desde la url de la web, ponemos el usuario *d'anne* nos dará un error SQL:

![[Pasted image 20260222171157.png]]

POdemos hacer la tipica SQLi para probar:

![[Pasted image 20260222172119.png]]

Y efectivamente hay una inyección SQL.  Hemos encontrado usuarios, asi que intentamos acceder a SSH con ellos:

![[Pasted image 20260222173100.png]]

Vemos el codigo fuente de la api que esta en python y tambien vemos un archivo de *Wireshark* que si lo geteamos haciendonos un servidor http rapido con python podemos observar:

![[Pasted image 20260222173217.png]]

Data FTP que s eha enviado, y vemos que es un usuario *root* y su contraseña *balulero*. Y con esto podemos acceder a la maquina como root:


