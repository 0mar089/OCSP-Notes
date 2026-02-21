
Empezamos con el escaneo:

![[Pasted image 20260214175919.png]]

Tenemos puerto 22 SSH abierto y puerto 80 HTTP abierto. 

ANtes de entrar a la web hacemos un *whatweb* para ver que tecnologias usa la pagina web:

![[Pasted image 20260214180028.png]]

No hay nada interesante, asi que entramos a la web para ver que tenemos:

![[Pasted image 20260214180110.png]]

La pagina web esta en blanco y lo unico que hay en el codigo fuente es esto. Asi que procedemos a hacer fuerza bruta de directorios. 

Al hacer fuzzing encontramos muchas rutas y hacemos fuzzing recursivo y encontramos esto:

![[Pasted image 20260214182642.png]]

Luego a esa ruta *javascript* le volvemos a hacer fuzzing

![[Pasted image 20260214182710.png]]

Y otra vez hasta llegar al codigo 200:

![[Pasted image 20260214182730.png]]

Y si entramos a ese endpoint, vemos que no es mas que la libreria de jquery asi que nada. 

Ahora intentamos hacer fuerza bruta con los dos usuarios que teniamos al servicio SSH. 


![[Pasted image 20260214183345.png]]

Vemos que hemos encontrado la constraseña asi que nos logueamos:

![[Pasted image 20260214183820.png]]

Encontramos el correo que nos envió el *Juan*, y nos da una contraseña. 

Accedemos al ssh otra vez como juan
![[Pasted image 20260214183941.png]]

Y si buscamos archivos ejecutados por el root, vemos:

![[Pasted image 20260214184020.png]]

Esto sigifnica que el usuario Juan puede ejecutar el interprete *Ruby* con máximos privilegios. 

![[Pasted image 20260214184109.png]]