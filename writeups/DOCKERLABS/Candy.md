
**IP**:  172.17.0.3
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 9 enero 2026
**Autor**:
**Descripción**: 

Web Enumeration
Fuzzing with gobuster
Information Leakage
Crypto Challenge - B64 Cipher
Joomla Enumeration
RCE Autenticado - Template Editing Abuse 
Enumeración del Sistema de Archivos
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - dd

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260109174457.png](../../_assets/603016d12db3d8d746889d7fd94b2b1cbef87420f5d5055d957a9a9c42c8e713.png)
Hay conexión 
TTL: 64 -> Linux

Hacemos un escaneo de puertos
![Pasted image 20260109174513.png](../../_assets/9fb306be5c41dddb0e89b7f6686376b009428d3c511c71f03d6aa5a09e3a7252.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo de versión al puertos específico 80. 
![Pasted image 20260109174742.png](../../_assets/ffcc55dd0d8e260b9ee70e96f2fc649ecd07de6745170de8477fa092950890f1.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260109174610.png](../../_assets/b385c6f7181a3a8c689b6410265f0744efd272beebf0592d081612acf2ce74b8.png)
Nos muestra un formulario de login.

Realizamos un fuzzing de directorios.
![Pasted image 20260109175234.png](../../_assets/c1b48208114cb6f4db538b640ca4e3d2289cfb0d43ce8a2207bc2d9cc14daf1c.png)
Y encontramos varios directorios .txt 

Nos dirigimos a `robots.txt`.
![Pasted image 20260109175411.png](../../_assets/14013c27f396b5bbbe7c70b30d73bbe480763cef9431ae1c1abbde62368532ab.png)
Y vemos una credenciales del usuario **admin**

Parece que esa contraseña esta codificada.

Nos dirigimos a dcode para decodearla.
![Pasted image 20260109175620.png](../../_assets/8141e93db57dfda520027527a2aa0038f548513b9f9e341bdf33c58b5e052160.png)
Vemos que esta en B64.

La decodeamos y vemos la contraseña
![Pasted image 20260109175709.png](../../_assets/3cc2d4ecb6ff66980123c14328d86c42d40b60309f66b24dc879d52336e40539.png)

### **Explotación**

Nos dirigimos a `/administrador`
![Pasted image 20260109180026.png](../../_assets/6038fbedc2afda13795e83db417c49b70cb371175d6f8e1c4efcbf45274506b0.png)

Y nos logeamos
![Pasted image 20260109180104.png](../../_assets/206fe1483e0a19a41fdde9f76ce09ddfec1942022c1ccda9e3baf38a277b5f95.png)
Logramos acceder con éxito al panel de administración.

En Joomla, la forma más rápida y común de conseguir una Reverse Shell siendo administrador es **editando las plantillas (Templates)** del sitio para inyectar código PHP malicioso.

Vamos a buscar un archivo `.php` que el sitio use y modificarlo.

Localizamos el Editor de Plantillas
![Pasted image 20260109181024.png](../../_assets/74035ded3ab63ef772e0337c11cad7ef9e34adfcd0cf31cde9025b3f783fe95c.png)


![Pasted image 20260109181140.png](../../_assets/269c1b393d7052d16344e96b546a8a7a426815eb97327e6fb12b2256e45b61f9.png)

Modificaremos el archivo error.php. Inyectamos nuestro payload de PHP al inicio del archivo.
![Pasted image 20260109182942.png](../../_assets/d7915fc4c3a10c25e9941cd3e540b2056992d95270e1a1d4809765032e84bda1.png)
Porque es el archivo que Joomla usa cuando alguien se equivoca de dirección (Error 404). Es muy fácil de activar intencionalmente.

Colocamos nuestro codigo malicioso.
![Pasted image 20260109183111.png](../../_assets/fa6291b1097e6b4c7cf2445f5629478a57531dc8ab784c42bfd13f1f7ccced4c.png)

Ponemos en escucha el puerto 4444.
![Pasted image 20260109183218.png](../../_assets/b58131f26fcee73e99cec195ca9dd65c2bc08a824c363cfc839f4d201f72ea3a.png)

A continuación, accedemos por navegador al archivo error.php.
```url
http://172.17.0.2/templates/cassiopeia/error.php
```

Y en nuestro puerto en escucha obtenemos un shell con el usuario `www-data`.
![Pasted image 20260109183342.png](../../_assets/1db2d62b1850e9e2cc38d307d4b4817389ebf39592994974c357ef5766b9d349.png)

### **Post-Explotación**

Viendo los usuarios en `/etc/passwd`, vemos que existe el usuario luisillo.
![Pasted image 20260109184048.png](../../_assets/547bb83d01d15afecb46165175bd05d7c588db556eb1895825f1c31b3215d2d6.png)

Hacemos una enumeración del Sistema de Archivos

Buscamos entre los directorios y acabamos encontrando, en /var/backups/hidden/, un archivo llamado `otro_caramelo.txt`
![Pasted image 20260109185336.png](../../_assets/af90e56f964d7c78a521020b67c9635fc34fee6c875c41adcc2a792a63ce833c.png)
Donde vemos un usuario y contraseña de luisillo.

Intentamos acceder al usuario `luisillo` con esas credenciales
![Pasted image 20260109190618.png](../../_assets/20471cf01ecc59bee149cd699007d69a67a983e1c0913e3b5ca13a99b9a3ad72.png)

Y logramos acceder
Luego listamos los permisos sudo
![Pasted image 20260109190642.png](../../_assets/c80b08146db5695f528d8068ce0311fda630a2ebaf535ee16a27b9875aaee42c.png)
 Vemos que podemos ejecutar `/dd` como **root** sin necesidad de una contraseña.
 
En gtfobins encontramos como escalar de privilegio a partir de `dd`.
![Pasted image 20260109190927.png](../../_assets/6893fc9dbc4e7fec8e6fa2d007dfd7be9fdf2ea21c39a91985f471700d79e4b0.png)
Seguimos los paso que indica:

Vamos a usar esa capacidad de escritura para agregar un usuario nuevo al archivo de usuarios (`/etc/passwd`) que tenga los mismos permisos que el jefe del sistema.

**$LFILE (Archivo a modificar):** Será `/etc/passwd`.
![Pasted image 20260109190820.png](../../_assets/7084362dbd17c469fb435703dbe71e03ea2aa2cab9b2b92c82c194a686ccac23.png)

**"data" (Lo que vas a escribir):** Será una línea de usuario con una contraseña que ya conocemos.
Inyectamos el comando.
![Pasted image 20260109191043.png](../../_assets/dc2e0d597fbc53604291359435be6d52f079681708f96851ec972699e39bf928.png)

Lanzamos un `shell` y lo obtenemos con permisos de root.
![Pasted image 20260109191137.png](../../_assets/8d4effe5380f9326eacd71a3f3b6fd9b2c91d33a143d6c2b50980c3a0e20aa4b.png)
Escalada exitosa a **root**.
