
Hacemos un escaneo al host y obtenemos:

![[Escaneo.png]]
Vemos que tiene un servicio web Apache corriendo, luego el servicio FTP con anonymous login activado y luego SSH. 

Primero accedemos por FTP para obtener esos archivos que se pueden obtener logueando como anonimo. 

![[Pasted image 20260214123231.png]]

Se puede leer posibles usuarios del sistema como **Gonza** o **Russoski** o tambien **Nágore**

Pasamos al lado de la web, y antes de acceder hacemos un *whatweb* para ver que tecnologias usa:

![[Pasted image 20260214123429.png]]

No usa nada nuevo, asi que accedemos a la web y vemos que podemos hacer:

![[Pasted image 20260214123546.png]]

Es una web de gimnasioy de cambiar tu cuerpo. Si bajamos mas abajo vemos que hay un formulario para pedir informacion o ayuda:

![[Pasted image 20260214123619.png]]

Al hacer el formulario esto hace una solicitud y envia la informacion:

![[Pasted image 20260214123717.png]]


Asi que de momento lo que podemos hacer es hacer fuerza bruta de directorios para ver que info podemos sacar:

![[Pasted image 20260214123809.png]]

Vemos que hay 2 rutas posibles con status code *Redirect*. Si intentamos acceder a cada una de ellas:

**important:**

![[Pasted image 20260214123942.png]]

Si entramos al archivo important.md encontramos algo con poca informacion:

![[Pasted image 20260214123926.png]]


**backup:**

![[Pasted image 20260214124022.png]]

![[Pasted image 20260214124033.png]]

Aqui ya tenemos algo de info, un usuario vàlido del sistema es russoski. Y aún no lo ha cambiado. 

Podemos hacer un ataque de fuerza bruta al servicio SSH para intentar descubrir la contraseña para conectarnos remotamente al servidor:

```
sudo medusa -M ssh -h 172.17.0.2 -u russoski -P /usr/share/wordlists/rockyou.txt -t 10
```

![[Pasted image 20260214131053.png]]

Ya tenemos acceso al servidor con bajos privilegios.


Si hacemos un ls de todo lo que russoski puede ver en su directorio del home podemos ver:

![[Pasted image 20260214133103.png]]

No hay mucha info, por lo tanto ahora podemos intentar escalar privilegios con los comandos basicos:

```
sudo -l
```

Este comando permite ver que comandos como superusuario podemos ejecutar siendo usuario russoski.

![[Pasted image 20260214133206.png]]

Vulnerabilidad encontrada, podemos ejecutar vim como usuario root. 


