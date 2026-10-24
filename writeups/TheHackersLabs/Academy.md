
**IP**:  192.168.20.139
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 23 diciembre 2025
**Autor**:
**Descripción**: 

Web Enumeration
Fuzzing con gobuster
Enumeración de WordPress con wpscan
File Manager WordPress Exploitation - Unauthenticated File Upload leading to RCE
Enumeración de procesos y comandos - Pspy
[[Escalada de Privilegios#**Escala de Privilegios mediante Cron Jobs]]

**Flags**:  


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un arp-scan para ver la dirección ip de la maquina.
![Pasted image 20260209173153.png](../../_assets/3a33cbcc498cd31b59e3f09e9265c9de9310f41bcd9ecdc8d70d9d7a761b2480.png)

Hacemos un ping:
![Pasted image 20260209173224.png](../../_assets/80d9b30cee112c49e8a946c3a3fa5e4b155f9299334459de1263fa67717ca0b8.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos TCP abiertos
![Pasted image 20260209173302.png](../../_assets/9db34e68afe511f3c29345ec1746b7ea270485d607458693f14f7dce8f418b8a.png)
Puertos abiertos: 
Puertos 22 (SSH) 
Puertos 80 (HTTP)

Escaneo de las versiones de los puertos específicos 22 y 80
![Pasted image 20260209173349.png](../../_assets/cc0b8a8c21e7c27a0b779a0364f795831ba7804521ca44fcbe7e6e5962f09cfc.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260209173445.png](../../_assets/03810e9f31ab0ad0a5dfa2694a6c8251e3808482ff0c692c24243317c0ce08ed.png)
El servidor respondió con la página por defecto de Apache2 en Debian

Realizamos un fuzzing de directorios.
![Pasted image 20260209173810.png](../../_assets/51ed59abf647e95069fd18e63daf1fddff0648a7abd9da783f92942f09704a06.png)
Vemos el directorio `wordpress`.

Y nos dirigimos a `192.168.20.139/wordpress`
![Pasted image 20260209173748.png](../../_assets/daacb1b0d25df945451186c14510709fc6d0baa40e436cbed17ed29d55a8e5f4.png)

Realizamos una enumeración con [[wpscan]]
![Pasted image 20260209175410.png](../../_assets/01156f4fbfce25c64ef7b7d9146b6e668f9ab62c14b6f2c4f04cadf0794b6c9e.png)
![Pasted image 20260209174234.png](../../_assets/e5f3682d536bf881a552d3174d24182e989a7deaa588e153342f23dbda98eb50.png)
Vemos un usuario identificado `dylan`.

Hacemos un un fuzzing de directorios a `192.168.20.139/wordpress`
![Pasted image 20260209174717.png](../../_assets/129f338f9d3453e651754296722d750efd9d18483d2ed7b998301faf4ecb9ffe.png)
Vemos los directorios donde podremos logearnos.

Realizamos una fuerza bruta a esa ruta con el usuario que encontramos **dylan** 
![Pasted image 20260209175425.png](../../_assets/87fc9f672a5f18232fec52e13df144bbf1a508fc9b190166f7c86e6fa831dc47.png)
![Pasted image 20260209175000.png](../../_assets/58f3294db8f65d60aea5c6e2ec658f93b53b66e991c969514da496eed96d0cdf.png)
Encontramos las credenciales de **dylan**

Y al dirigirnos  a `192.168.20.139/wordpress/wp-admin` no logramos acceder y nos redirige al dominio `academy.thl`.
![Pasted image 20260209175926.png](../../_assets/679e068c04e8bb8c6d09bda90cc2b7a1105d5c835a99c2aa7813faef5b6ee1a1.png)

Agregamos aquel nombre de dominio a `/etc/hosts`
```bash
sudo nano /etc/hosts
```

Y accedemos con el usuario **dylan**
![Pasted image 20260209180545.png](../../_assets/b919d9938f67399a186236c83ab80ab08a7ac1dd65331bc090e0fc0fb5a0dd06.png)

### **Explotación**

Al entrar podemos ver que podemos subir archivos
![Pasted image 20260209181706.png](../../_assets/0afea678972cc550333293814ca6f135be89c5b42976c697b4f25350785c513d.png)


Creamos nuestro archivo malicioso.
```bash
nano shell.php
```

Y lo subimos
![Pasted image 20260209181736.png](../../_assets/cd18c0cabf362ff13106ee7079a9e386f741c2af997226e4732a87ab713f5e0b.png)

Ponemos el puerto que asignamos en el archivo malicioso php.
```bash
nc -nvlp 4545
```

Nos dirigimos a
```
http://192.168.20.139/wordpress/shell.php
```

Y logramos obtener la Reverse Shell
![Pasted image 20260209182132.png](../../_assets/b62d246d28b8c8afa25382cc52edfc39f67eaaa627d055e592ed39ac027b9cdf.png)

### **Post-Explotación**

Para poder escalar privilegio, después de revisar varias alternativas
Descargamos `pspy64` para ver los procesos en tiempo real
![Pasted image 20260209182936.png](../../_assets/12524070536c003b974711babd0eb3129b2f4f7a9209c601a57f7b61b0b2f093.png)

Y vemos que hay un proceso con el UID=0 que ejecuta `/bin/sh -c /opt/backup.sh`.
![Pasted image 20260209183032.png](../../_assets/b741e0e4e1a1f001eed9b0dbe7e050749a9df091a039123b4a0e50b9d045a2c5.png)

Nos dirigimos a `/opt` y vemos que solo hay `backup.py` mas no `backup.sh`.
![Pasted image 20260209183239.png](../../_assets/1a6da44d1f7b8dbc1db6e71434dcb9f5bdb3e81e6062058ec81b39e0f0a14b13.png)

Vemos que tenemos permisos de escritura del directorio `/opt`.
![Pasted image 20260209184051.png](../../_assets/29d88d5ae6e36741e9c9dd655311902a73b8afb5ce234f498576dde54daa9a88.png)

Entonces procedemos a crear `backup.sh` enviando una shell remota a nuestra maquina atacante hacia el puerto `4444`.
![Pasted image 20260209185039.png](../../_assets/1687ebcedce829630223b343bcb5094aec7cb45326cf628799ba283650c52396.png)

Le damos permisos de ejecución
```bash
chmod +x backup.sh
```

Ponemos el puerto `4444` en escucha
```bash
nc -nvlp 4444
```

Y después de un minuto el `bakcup.sh` se ejecuta y nos da una shell remota con permisos de **root**
![Pasted image 20260209185119.png](../../_assets/1e2c28657f6c67e746b38da5f25c9d63174122dcce84e2541b1d72f780182439.png)

