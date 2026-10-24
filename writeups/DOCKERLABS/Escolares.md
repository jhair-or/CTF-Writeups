
**Source:** DockeLabs
**S.O**: Linux
**Level:** Fácil
**Date:** 17 enero 2026
**Autor**:
**Descripción**: 

Enumeración Web
Information Leakage
Enumeración de WordPress con wpscan
Generación de contraseña personalizadas - CUPP.PY
Fuerza bruta con wpscan
File Manager WordPress Exploitation - Unauthenticated File Upload leading to RCE
Enumeración del Sistema de Archivos
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - awk

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260117163304.png](../../_assets/74c4a9de969bc32c522708653c9d28267a5c331e436b0f7b4a0f9d6f4f213fdb.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260117163340.png](../../_assets/0354ab0212dbf03f9bd8b0bac7501e15552d7750a9021dd892f0570185c75313.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

Escaneo de versión al puertos específicos.
![Pasted image 20260117163355.png](../../_assets/bd8572260176a546061e858e0a8aa955ec231d9d7bae91e6a061ca1b50653753.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260117163423.png](../../_assets/5f78b77d626b742eda5e537da3a5cd22ecc4cf33966e44ae307af7a0296ce3ec.png)

Inspeccionando mas la pagina web vemos una sesión de los nombre de profesores y el nombre del admin del wordpress
![Pasted image 20260118183125.png](../../_assets/ca72b15e64f08b8c0656767656e82f60e6021d80516aab55d6425f30df8f98bb.png)

Realizamos una enumeración con [[wpscan]]
```bash
wpscan --url http://172.17.0.2/wordpress --enumerate u,vp,vt
```
![Pasted image 20260118185222.png](../../_assets/6577baff11e03873a3749d988c29614fd270904b52bbe3019d501a147f58dd56.png)
![Pasted image 20260118185207.png](../../_assets/bba11a8028a0b4783ab44cdf6acf17b4207c63c9c3ba6139bb8ce480bc3b692b.png)
Y encontramos el user del admin del wordpress luis.

### **Explotación**

Ahora para encontrar su contraseña realizaremos una fuerza bruta.
De ello crearemos una lista de contraseñas de acuerdo a toda la información encontramos sobre Luis en la pagina de wordpress 
Para ello utilizaremos la herramienta `cupp.py` para que nos genere la lista de contraseñas con los datos encontrados como su fecha de compleaños, su nombre, carrera, etc
![Pasted image 20260118190329.png](../../_assets/4be0367084ab7b0879b2c20d3b06a95e12e33f34764d8f9c3b9a21d9c157f570.png)
Obtenemos la lista de contraseñas llamadas `luis.txt`

Procedemos a realizar con `wpscan`.
```bash
wpscan -U luisillo -P luis.txt --url 172.17.0.2/wordpress
```
![Pasted image 20260118191350.png](../../_assets/6f10d8a1a45d678ce5a3d1ffefd175f5d61a18fd3ffd6bb2d4196fa344d19dc4.png)
Logramos encontrar la contraseña

Nos dirigimos a 
`http://172.17.0.2/wordpress/wp-login.php`
![Pasted image 20260118194603.png](../../_assets/3168d932e959a5f800c16dd8d079dd987b16db2bf3309f203565fc5ed7b5a503.png)
Accedemos

Pero nos redirige a `escolares.dl`
![Pasted image 20260118194738.png](../../_assets/dad30b99e0bbdd106fc470730876c954490c67d383fb4fe979634c425f3ee2b8.png)

Declaramos el dominio `escolares.dl` en `/etc/hosts`
![Pasted image 20260118195030.png](../../_assets/02c97caebc383bc8bfb91bfc1d7bd5a8c69fb4ca4258fe52837a0b595b9b4aab.png)

Y nos dirigimos a aquella ruta para logearnos
![Pasted image 20260118195059.png](../../_assets/56c0f4ad839c463fe921e297a073a7fcdb3bcd416b3d520109c01a9473d999c6.png)


Y logramos acceder
![Pasted image 20260118195235.png](../../_assets/1e99bd73d2f8cd67e841122382e46feca0ff219fc3625a45316a8a534ffbf7a8.png)

Una vez accedido al panel nos dirigimos a la parte de `WP File Manager`.
![Pasted image 20260118195418.png](../../_assets/aaa8edd2aa7c870d9eefdf30cbe4933f5c37dd9e162d837b9e8e81be52f211a8.png)

Y creamos el archivo malicioso que nos dara una reverse shell
![Pasted image 20260118195707.png](../../_assets/57a07907900d941bdfa469f713c0f44b8bc453afc8ea147974b61dfb87acbd53.png)

Lo subimos
![Pasted image 20260118195821.png](../../_assets/ab8dbfde9487d1d48d90730b3ac763c63d587aa7994982a614864850ab2e363a.png)

Ponemos el puerto que asignamos en escucha
![Pasted image 20260118195900.png](../../_assets/8cf984364138eb5e9fee14324eeb4b307999ecfb022fb3e5b2a1d81eb44ff3c2.png)

Nos dirigimos a:
```
http://escolares.dl/wordpress/wp-admin/php-reverse-shell.php
```

Y logramos obtener una sesión  como el usuario `www-data`
![Pasted image 20260118200015.png](../../_assets/c44c5a5e2ceb42f130d4643b9e8bbca2026da967d95530e50df9cf5657f7cfe0.png)

Hacemos una tratamiento TTY
![Pasted image 20260118200148.png](../../_assets/a3b0d2dbd127cb5906b6f31c4d866a3b2790e628167997210e38b58262bffba0.png)

### **Post-Explotación**

Nos dirigimos al directorio `/home`
![Pasted image 20260118200233.png](../../_assets/e4831ce1fea5ebba7819ec46743a85a5241606a1a2579e2107d5369a56fb7a7a.png)
Y vemos un archivo llamado `secret.txt`.

Lo leemos
![Pasted image 20260118200239.png](../../_assets/a708fdda2425ae0561f9cf9c8ba12d52591d919750b6d9d6236b94f3688af806.png)
Y nos dan al parecer una contraseña

La probamos con el usuario `luisillo`
![Pasted image 20260118200320.png](../../_assets/d99258ed1c7d29b0520e7646fbb66147cc80c4188c95949d4d66d89875fc1d6d.png)
Y logramos ser el usuario `luisillo`.

Listamos los permisos sudo
![Pasted image 20260118200339.png](../../_assets/063471fca34a58447f33c21f40b797940775b1ed67620c3f0195ea82aa962303.png)
Y vemos que podemos ejecutar `awk` como **root** sin necesidad de una contraseña. 

Ejectuamos el siguiente comando para escalar al usuario **root**
```bash
sudo /usr/bin/awk 'BEGIN {system("/bin/sh")}'
```
![Pasted image 20260118200804.png](../../_assets/90ac8b2d99a7b09d134be19d63fe1615bbdbc7d90d494efca2f6fb9ec434518a.png)
Escalada exitosa a root.
