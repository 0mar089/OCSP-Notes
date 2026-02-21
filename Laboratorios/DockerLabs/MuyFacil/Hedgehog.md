Empezamos el escaneo *nmap*:

![[Pasted image 20260218193032.png]]

Puerto 80 HTTP y puerto 22 SSH abiertos.

Empezamos con el escaneo web con lo basico, *whatweb, fuzzing...*:

![[Pasted image 20260218193109.png]]

No hay mucha tecnologia que s epueda explotar, asi que hacemos un curl para ver la pagina web que tiene:
![[Pasted image 20260218193202.png]]

Solo pone tails. Hacemos Fuzzing para ver si hay algun recurso interesante:

![[Pasted image 20260218193241.png]]

Tampoco hay algo interesante en el fuzzing, asi que haremos un ataque de fuerza bruta a SSH usando tails como nombre. Pero usaremos la wordlist *rockyou* pero invertida:

```
tac rockyou.txt > rockyou_invertido.txt
```

Y ademas le quitamos los espacios y eso:

```
sed -i 's/ //g' rockyou_invertido.txt
```

Y vemos que hemos encontrado una credencial válida:
![[Pasted image 20260218193506.png]]

Para escalar los privilegios, miramos los binarios que puede ejecutar cada usuario:

```
sudo -l
```

![[Pasted image 20260218194223.png]]

Significa que puedes usar comandos de usuario sonic sin password.

![[Pasted image 20260218194307.png]]

Y si intentamos acceder al usuario root:

![[Pasted image 20260218194326.png]]

