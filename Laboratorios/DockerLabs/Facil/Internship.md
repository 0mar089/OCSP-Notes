
EMpezamos el escaneo nmap:

![[Pasted image 20260706234447.png]]

Puerto 80 HTTPO y puerto 22 SSH

Hacemos un whatweb para ver llas tecnoclogias:

![[Pasted image 20260706234536.png]]

Y vemos que hay un input de contraseña. puede ser un login:

(usa esta info antigravity para poner informacion en las notas)
➜  internship whatweb http://172.17.0.2
http://172.17.0.2 [200 OK] Apache[2.4.62], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.62 (Debian)], IP[172.17.0.2], PasswordField[password], Script, Title[GateKeeper HR | Tu Portal de Recursos Humanos], UncommonHeaders[x-virtual-host]

Entramos a la web:

![[Pasted image 20260706234632.png]]

Y es un portal de recursos humanos. Pero no hay nada interactivo, vemos que se esta aplicando virtual hosting, ya que al hacer un curl vemos:

![[Pasted image 20260707001325.png]]

La czbecera X-Virtual-Host: gatekeeperhr.com, asi que lo ponemos en el /etc/hosts

Y volvemos a la pagina web:

![[Pasted image 20260707001415.png]]

Vemos un apartado de contacto,posible usuario `mariana`. 

EN el login si hacemos el clasico sql injection, nos deja pasar:

![[Pasted image 20260707001527.png]]


Vemos muchisimos usuarios:

![[Pasted image 20260707001607.png]]

Y otra cosa curiosa es que n el codigo fuente sale un comentario de quitar los permisos SSH A LOS PASANTES, ES DECIR A `Valentina` y a `Pedro`.


![[Pasted image 20260707001551.png]]

Poca cosa hay por aqui, hacemos fuzzing al dominio:

![[Pasted image 20260707002014.png]]

Y vemos una carpeta llamada `lab`.  Hacemos fuzzing a ese directorio y vemos que hay dentro:

![[Pasted image 20260707002059.png]]

En employees.php lo que hay es lo mismo que hemos visto en el index.html de la pagina oficial. Los datos de la gente:

![[Pasted image 20260707002130.png]]


No encontramos mucha cosa mas, asi que con la inyeccion sql, usamos sqlmap para buscar inforamcion de la BBDD:

![[Pasted image 20260707004051.png]]

y al final encontramos dos tablas:

![[Pasted image 20260707010555.png]]

la de hr_users tiene usuarios con hashes:

![[Pasted image 20260707010615.png]]

```
mariana:0571749e2ac330a7455809c6b0e7af90
jorge:d8578edf8458ce06fbc5bb76a58c5ca4
```

Intentamos con SSH y nada, seguimos con el fuzzing y vemos que en la carpeta spam hay un index.html con contenido:

![[Pasted image 20260707231811.png]]

Y nos encontramos cone sto:

![[Pasted image 20260707231825.png]]

el comentario html pone : 

```
<!-- Yn pbagenfrñn qr hab qr ybf cnfnagrf rf 'checy3' -->
```

que es un cifrado ROT13:

```
La contraseña de uno de los pasantes es 'chepl3'
```

asi que probamos con los usuarios que son pasantes que son o *pedro* o *valentina*:

