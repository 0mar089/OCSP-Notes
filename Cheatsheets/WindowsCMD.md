# COMANDOS DE WINDOWS (CMD)

Guía de comandos de consola de comandos (cmd.exe) estructurada por categorías de uso frecuente para administración, redes y pentesting.

---

## Navegación y Gestión de Archivos

| Comando      | Descripción                                       | Ejemplo de Uso                   |
| :----------- | :------------------------------------------------ | :------------------------------- |
| dir          | Lista archivos y carpetas del directorio actual.  | dir /a /od                       |
| cd           | Cambia el directorio de trabajo.                  | cd C:\Users                      |
| mkdir / md   | Crea una nueva carpeta.                           | mkdir NuevaCarpeta               |
| rmdir / rd   | Elimina una carpeta.                              | rd /s /q Carpeta                 |
| copy         | Copia uno o más archivos.                         | copy archivo.txt C:\Destino      |
| xcopy        | Copia archivos y árboles de directorios.          | xcopy /s /e C:\Origen C:\Destino |
| robocopy     | Copia avanzada y robusta de archivos/directorios. | robocopy Origen Destino /mir     |
| move         | Mueve archivos y renombra directorios.            | move *.txt C:\Destino            |
| del / erase  | Elimina uno o más archivos.                       | del /f /q temporal.txt           |
| ren / rename | Renombra un archivo o directorio.                 | ren viejo.txt nuevo.txt          |
| type         | Muestra el contenido de un archivo de texto.      | type config.json                 |
| attrib       | Muestra o cambia los atributos de los archivos.   | attrib +h +r secreto.txt         |
| tree         | Muestra gráficamente la estructura de carpetas.   | tree /f                          |
| where        | Localiza archivos que coinciden con un patrón.    | where java                       |

---

## Red y Conectividad

| Comando | Descripción | Ejemplo de Uso |
| :--- | :--- | :--- |
| ipconfig | Muestra la configuración de red IP. | ipconfig /all |
| ping | Envía paquetes ICMP para probar la conexión. | ping 8.8.8.8 -t |
| tracert | Muestra la ruta que siguen los paquetes a un host. | tracert google.com |
| nslookup | Realiza consultas a servidores DNS. | nslookup google.com |
| netstat | Muestra conexiones activas y puertos abiertos. | netstat -ano |
| arp | Muestra y modifica las tablas de traducción IP-MAC. | arp -a |
| route | Muestra o modifica las tablas de enrutamiento. | route print |
| netsh | Configuración avanzada de interfaces y firewall. | netsh advfirewall show currentprofile |

*Para ver contraseñas WiFi guardadas:*
```cmd
netsh wlan show profile name="NombreWiFi" key=clear
```

---

## Información y Control del Sistema

| Comando | Descripción | Ejemplo de Uso |
| :--- | :--- | :--- |
| systeminfo | Muestra datos detallados del OS y hardware. | systeminfo |
| hostname | Muestra el nombre de la máquina. | hostname |
| ver | Muestra la versión del sistema operativo. | ver |
| tasklist | Lista todos los procesos en ejecución. | tasklist /svc |
| taskkill | Finaliza procesos por nombre o PID. | taskkill /pid 1234 /f |
| sc | Administra y consulta servicios del sistema. | sc query state= all |
| query | Consulta el estado de sesiones y usuarios. | query user |
| whoami | Muestra el usuario actual y sus privilegios. | whoami /priv |
| gpresult | Muestra el conjunto de directivas de grupo (GPO). | gpresult /r |
| wmic | Interfaz de línea de comandos de WMI (obsoleto pero útil). | wmic product get name |

---

## Usuarios, Recursos y Permisos

| Comando | Descripción | Ejemplo de Uso |
| :--- | :--- | :--- |
| net user | Administra cuentas de usuario locales. | net user Administrador /active:yes |
| net localgroup | Administra grupos locales. | net localgroup Administradores |
| net share | Crea, elimina y gestiona recursos compartidos. | net share |
| net use | Conecta o desconecta recursos compartidos de red. | net use Z: \\192.168.1.5\Compartido |
| icacls | Muestra o modifica listas de control de acceso (ACLs). | icacls archivo.txt /grant Usuario:F |
| runas | Ejecuta una aplicación con credenciales de otro usuario. | runas /user:Administrador cmd.exe |

---

## Mantenimiento y Configuración de Discos

| Comando | Descripción | Ejemplo de Uso |
| :--- | :--- | :--- |
| chkdsk | Verifica y repara errores en el disco. | chkdsk C: /f /r |
| sfc | Comprueba y repara archivos dañados del sistema. | sfc /scannow |
| dism | Repara y prepara imágenes del sistema. | dism /online /cleanup-image /restorehealth |
| diskpart | Utilidad avanzada de gestión de particiones. | diskpart |
| format | Formatea una unidad de disco. | format F: /fs:ntfs /q |
| cipher | Cifra directorios o limpia espacio libre en disco. | cipher /w:C:` |
| assoc | Muestra o modifica asociaciones de archivos. | assoc .txt |
| ftype | Define los programas para abrir tipos de archivos. | ftype txtfile |

---

## Control de Energía y Apagado

| Comando | Descripción | Ejemplo de Uso |
| :--- | :--- | :--- |
| shutdown | Apaga, reinicia o cierra sesión en el equipo. | shutdown /r /t 0 |
| powercfg | Configura opciones de energía del sistema. | powercfg /hibernate off |

---

## Trucos y Atajos Útiles en CMD

- **Limpiar pantalla:** cls
- **Salir de CMD:** exit
- **Copiar salida directa al portapapeles:** ipconfig | clip
- **Crear un archivo en blanco rápidamente:** type NUL > archivo.txt o echo. > archivo.txt
- **Cambiar color de la consola:** color 0a (Verde, tipo matriz), color 07 (Restaurar)
- **Consultar la ayuda de cualquier comando:** comando /? o help comando
- **Crear un alias temporal (como en Linux):** doskey ls=dir
