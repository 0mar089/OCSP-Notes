
Hacemos el escaneo nmap:

![[Pasted image 20260813030601.png]]

Vemos que hay el puerto 22 SSH abierto, 80 HTTP, 139 SMB y 445 SMB

Empezamos por el puerto 80, ya que el puerto 22 SSH no es vulnerable a enumeración de usuarios. 

Hacemos un whatweb:
![[Pasted image 20260813031245.png]]

NO hay mucha tecnologia a parte de la versión de Apache. 

Entramos a la web:

![[Pasted image 20260813031333.png]]

Hay un login.  Intentamos hacer un sql inyeccion clasica peor nada, ahcemos fuzzing ahora:

![[Pasted image 20260813032214.png]]

Encontramos **productos.php**




