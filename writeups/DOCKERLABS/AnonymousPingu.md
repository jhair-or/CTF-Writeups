
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 1 febrero 2026
**Autor**: El Pingüino de Mario
**Descripción**: 

Web Enumeration
ftp Enumeration
Remote Code Execution (RCE) vía Arbitrary File Upload en ftp
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] con man, dpkg, chown

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260201161959.png](../../_assets/e3830d9f102cd1cb9c5c6208d49445617b8feee2c3033f4e393b058b18dcbd67.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260201162032.png](../../_assets/5ec085378cfcef77e2c318db8a65b8386580133a3d1bd1fe4507aeddb961f7f0.png)
Puertos abiertos: 
Puertos 21 (FTP)
Puertos 80 (HTTP)

Escaneo de versión al puertos específicos 21,80.
![Pasted image 20260201162210.png](../../_assets/d74ba41291f7d2379c5808f1418380285afddd9697e48f84c15b4b5ebf29e452.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260201162327.png](../../_assets/be6ec6b800f00fdc533786a697f09c554570d22942f3a1740b88e8e52a539958.png)
No vemos nada interesante 

Hacemos un fuzzing de directorios.
![Pasted image 20260201163546.png](../../_assets/f1c1f3b63bf77e485bdeb28c2d3eb43306a4670cd63fd4bc79b71c9d408e7918.png)
Vemos que hay una ruta llamada `/upload`

Hacemos una enumeración al ftp.
![Pasted image 20260201163208.png](../../_assets/638877b95b1f61f60be6de668cae3eb0a49533c581d128edea1e3e0c22287096.png)
Nos logeamos con usuario **Anonymous**
Vemos que el directorio al que se accede por FTP es el mismo directorio que sirve la página web

### **Explotación**

Entonces nos dirigimos a la carpeta `upload`
![Pasted image 20260201164507.png](../../_assets/076bd1c2f396a71c2548e37f8c1c580b537b29a4b9e67339a7438f124bf7cbb2.png)
Y subimos un archivo malicioso en php `php-reverse-shell.php` que nos dará una Reverse Shell

Nos dirigimos a `/uploads` para verificar que se subió.
![Pasted image 20260201164534.png](../../_assets/5ede9ba2281c6770c77cab28d439267c8319bf219d1dd6ff6d67d7a462423c76.png)

Ponemos nuestro puerto que asignamos en `php-reverse-shell.php` en escucha que al ejecutar `php-reverse-shell.php` no logra dar la shell interactiva con el usuario `www-data`.
![Pasted image 20260201164729.png](../../_assets/e0658794ddd7e91af2550d819dc8a2e1902341ba10106b9f992073033652d2e6.png)

### **Post-Explotación**

Listamos los permisos sudo
![Pasted image 20260201165031.png](../../_assets/2b51d16e511e197f0e38cd0d17d7873f9c796e5bc6ca9868bce1e6a269726b00.png)
Veos que podemos ejecutar `/man` como el usuario`pingu` sin necesidad de una contraseña.

Según gtfobins ejecutaremos al ejecutar el siguiente comando
```bash
sudo -u pingu /usr/bin/man man
```

Una vez que estés dentro de la pantalla del manual escribiremos lo siguiente
![Pasted image 20260201181645.png](../../_assets/b76c9114431523a0e18294b3e63fa293d3fce20170929567bccbe57a9cedc9ce.png)
Ahora somos el usuario `pingu` 

Listamos los permisos sudo
![Pasted image 20260216191020.png](../../_assets/d1743cec02aa550db9d1d97b73249d6e3bfddeaaaef13bbf1932023208b578a4.png)
Vemos que podemos ejecutar `/dpkg` como gladys sin necesidad de una contraseña 
El proceso con `dpkg` es idéntico que al de `man` para escalar privilegios.

Ejecutamos en siguiente comando
```bash
sudo -u gladys /usr/bin/dpkg -l
```

Una vez ejecutado, escribimos esto
![Pasted image 20260201182234.png](../../_assets/8534446870c5c1cd95cc07d11cd4c0d767667b1c0a5ea75744488e629e7ce4c5.png)

Y logramos acceder como el usuario gladys.
![Pasted image 20260201182507.png](../../_assets/9764e717983c478c865ed1b2fa15a1a1af50b79c1ebbc411aa5ececfeaa1437c.png)
Y vemos que podemos ejecutar `/chown` como **root** sin necesidad de una contraseña.

`chown` permite adueñarte de cualquier archivo del sistema.

Entonces nos adueñaremos del archivo `/etc/passwd` para modificar la contraseña.
![Pasted image 20260201182706.png](../../_assets/9207da35f729212dd2fcb487a9d7298c3d256ffeca6e8868ae5ee88a1187773e.png)
Ahora somo los dueños de `/etc/passwd`

Creamos un usuario `hacker` sin contraseña con privilegios de root 
```bash
echo "hacker::0:0:root:/root:/bin/bash" >> /etc/passwd
```

Accedemos como `hacker` y logramos escalar al usuario **root*
![Pasted image 20260201183541.png](../../_assets/82df33fea23674ba917ca0194502378b4799e7ab90da724ab392dfdf4556e335.png)
