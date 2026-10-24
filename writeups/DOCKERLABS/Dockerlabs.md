
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 16 Enero 2026
**Autor**:
**Descripción**: 

Web Enumeration
Fuzzing con gobuster
Bypass -  Abusing File Upload (Content-Type Manipulation + PHAR extension) [RCE]
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - grep, cut

**Flags**: 


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260116140211.png](../../_assets/5108045a6f87131a24b624ea782079a1a0aa2a8a22ab03ddbc17ccd535ca536d.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260116140317.png](../../_assets/ee5fd7e49ffa201300e63e43e12f117ab98b47c172c30199b208eeb70f00c90c.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo de versión al puertos específico 80. 
![Pasted image 20260116140348.png](../../_assets/f4d76ac7f497ff7a625f888c65629f8cd1b0b0cd9fea1a7eaf34cbf53244535d.png)


Accedemos a la pagina web
![Pasted image 20260116140336.png](../../_assets/21ce3841525b2aaf78b6c0b1333e57cd984bcd8d7efad111392e1cd848513a41.png)

Hacemos un fuzzing de directorios.
![Pasted image 20260117143718.png](../../_assets/6484c461734d78ae1a90023f91b8cd7053d17a4bec313a3f20ef01cc9de1701e.png)

Nos dirigimos a `machine.php`
![Pasted image 20260117143925.png](../../_assets/bbb355748f262e50998560b0b66c58a7ce144fd221b3442e9a2c5b0f79861353.png)
Y vemos que podemos subir archivos

Creamos nuestro archivo con código maliciosos que nos dara una reverse shell
![Pasted image 20260117143839.png](../../_assets/259cfcf9ea9fdedf86203cf3f8d6c739b0f159d7a125b4338ae528e2ab437700.png)

Subimos el archivo
![Pasted image 20260117143901.png](../../_assets/fedf94f67307b7cc99a50e369fb23c685363d73c322ef3c86e18bf653d57535a.png)

Y al subirlo nos dice esto:
![Pasted image 20260117143941.png](../../_assets/c13aedc4b59f7c3e37a1cff45e5ecbeabadd6229a75b0953bf399665dc112b06.png)

Entonces lo Bypaseamos por una variante a `.phar`
```bash
mv php-reverse-shell.php php-reverse-shell.phar
```

Y logramos subir el archivo correctamente.
![Pasted image 20260117145938.png](../../_assets/e519d64a1b530822bddb0abd2e24d9dccab7f17f9962f747f4172832e9b035f2.png)

Ponemos el puerto `4545` en escucha
```bash
nc -nlvp 4545
```

Nos dirigimos al directorio de `/uploads`
![Pasted image 20260117150022.png](../../_assets/7d15dace16134fd60855feeb27d116fcc985d72290674124101f00ffb90fac72.png)
Lo ejecutamos

Y en nuestra terminal recibimos la revese sheell
![Pasted image 20260117150034.png](../../_assets/15dd1bd0c733a58487aab51a54054c4a659504ed23fecdb29f28d4985f17664b.png)

Listamos los permisos sudo
![Pasted image 20260117150153.png](../../_assets/d3f0b8468c0603e6651ecb098c8ee7cc652424a06be1ab949fc2e83abda8ed58.png)
Y vemos que podemos ejecutar `cut` y `grep` como **root** si necesidad de una contraseña.
Estos ejecutables puede leer datos de archivos locales.

Dado que tiene permisos podremos leer archivos como `/etc/shadow`.
![Pasted image 20260117151845.png](../../_assets/ca9f00e875c2e53ce57a4b52682df67c7333957f8a902d554e9bd26f0dbb8bcd.png)
Copiamos la clave hasheada de `dbadmin`.

Y lo crakeamos con `john`
```bash
john --wordlist=/usr/share/wordlists/rockyou.txt --format=crypt hashes.txt
```
![Pasted image 20260117151820.png](../../_assets/880091c11b53ccd04c0b94f1f474915722263f09ae6b631af65c5bf5e5c22a62.png)
Logramos crakear obtener las credenciales

Accedemos a `dbadmin`
![Pasted image 20260117152155.png](../../_assets/95b4b1b8a2bc630caab1fd59024ef8eda4452824b502da28af225a99f78110fe.png)

Enumeramos el sistema de archivos
Y dentro del directorio `/opt` vemos que hay 
![Pasted image 20260117152338.png](../../_assets/e49fa4408a3101e8932c8c149fb74e6f0b9744e1b5f048ae13e038b365639537.png)

Leemos `nota.txt`
![Pasted image 20260117152400.png](../../_assets/a028c9d8836b213c651cd66fb354d83568c9516162f464e3714b9866d16f6083.png)
Y nos indica donde es que se ubica la contraseña del usuario **root**
La `clave.txt` se encuentra dentro del directorio root.

Volvemos al usuario `www-data` donde podemos usar cut con privilegio de root.
Entonces
```bash
sudo -u root /usr/bin/cut -d "" -f1 "/root/clave.txt"
```
![Pasted image 20260117152917.png](../../_assets/863d41193373cb78ab8b95139a2ec6b448db32522cbe39b7c28654a490852b30.png)
Y obtenemos la contraseña de **root**

Escalada exitosa a root.
![Pasted image 20260117153005.png](../../_assets/95f40d8489a7d34620ca3281d47d30a8e0c9bdfa83303ac50cffa80f75c02e77.png)

