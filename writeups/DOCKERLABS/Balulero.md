
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 15 enero 2026
**Autor**: El Pingüino de Mario
**Descripción**: 

Enumeración Web
Fuzzing de directorios con gobuster
Information Leakage
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] con php
Enumeración de procesos y comandos - Pspy
[[Escalada de Privilegios#**Escala de Privilegios mediante Cron Jobs]]

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260114144449.png](../../_assets/52c7b6442ec1468771ef3429e16b9bcd87c0567f2447bb8736d5a7bcc0e33a8f.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260114144537.png](../../_assets/3985313e918506c06725c4fccb1078e8fde98fc458274a55283793258eaad752.png)
Puertos abiertos: 
Puerto 22 (SSH)
Puerto 80 (HTTP)

Escaneo de versión al puertos específico 22 y 80
![Pasted image 20260114144547.png](../../_assets/40794d2ce078e63a610a9604e6c5442635a372407651e50ed21da46c2b552230.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260114144847.png](../../_assets/6bcb6570f8d25db03060f87e9868bf08e36a3e230e322544373b85ee823f5430.png)
Y solo vemos el un gif de un perro

Hacemos un fuzzing de directorios.
![Pasted image 20260115211429.png](../../_assets/6b6067babb5b8f92b3b6ec4c6656b8449935e263d47970cf477aab7ea0c91509.png)
Y vemos una ruta interesante llamada `/script.js`.

Accedemos a aquella ruta
![Pasted image 20260115211616.png](../../_assets/93f5883e933e8cd932413af784ed8dc015d8ccdd65b0962a2d285e54b34f255b.png)
Y vemos que dice que hay un archivo donde se guarda el backup de la contraseña, y es `.env_de_baluchingon`.

Nos dirigimos ahi
![Pasted image 20260115211922.png](../../_assets/baae96a013e9536e4a347067fcaf02eada783686771c6fb99f0fc099e09662ca.png)
Y vemos las credenciales de usuario `balu`

Ingresamos dichas credenciales al servicio SSH
![Pasted image 20260115212507.png](../../_assets/adacbb407feee97391d74a43c6a9695733bf6d8c8942dd2a3859c893519e27f1.png)
Y logramos acceder como el usuario `balu`.

Listamos los permisos sudo.
![Pasted image 20260115212637.png](../../_assets/bbd73eeeeaccf25a54445dfbb1ae1269c0228bac394fe210089c327b6b110bb0.png)
Vemos que podemos ejecutar `php` como el usuario `chocolate` sin necesidad de una contraseña.

Lo ejecutamos invocando una consola como usuario el usuario `chocolate`
```bash
sudo -u chocolate /usr/bin/php -r 'system("/bin/bash");'
```
![Pasted image 20260115213024.png](../../_assets/7b28cedf6a6080055a8cc8de767af41b40a1c196cc105631fdf601cba3b8ef0d.png)
Y logramos ser el usuario `chocolate`

Para poder escalar privilegios siendo el usuario `chocolate` 
Utilizaremos la herramienta `pspy64` 
Abriremos un servicio web para enviar el `pspy64` a la maquina victima.
![Pasted image 20260115224834.png](../../_assets/37a3ab2bddf5f06126a41178d710539b75bb6d7b90ecc8cea2e3e0190b9fbcfb.png)

En la sesión de `chocolate` no dirigimos a `/tmp` y lo descargamos con `wget`
![Pasted image 20260115224901.png](../../_assets/6f3d339b9828a2da7dee8d2bb3cabccf24fbc952aa6f242623a722e92531f812.png)

Le damos permisos de ejecución.
![Pasted image 20260115224913.png](../../_assets/37f04e233ef302e7f55098c6adbf8da79a24b314aa7c10686403ae6672b980c1.png)

Lo ejecutamos
![Pasted image 20260115225009.png](../../_assets/7fea7579bbb929f0b5d081e7e53fd86f9e27e37a4fe91660142f7847fd416370.png)
Al ejecutarlo
Vemos que el archivo `/opt/script.php` se ejecuta como root cada 5 segundos.

Vemos que tenemos permiso de escritura al ser parte del grupo **chocolate**
![Pasted image 20260115225536.png](../../_assets/da03354716639ddbc08df7a0f8028f5fdf6e738240f5e12b8153dd7414709f46.png)

Entonces modificamos el script y hacemos que le de permisos SUID a `/bin/bash`.
```bash
echo "<?php system('chmod u+s /bin/bash'); ?>" > /opt/script.php
```
![Pasted image 20260115225644.png](../../_assets/0f2abd2d36eab493ced2f4b441bddabbd54f3b610e91d793929fb26256dd7dcb.png)

Despues de 5 segundos pasados, vemos que se ejecuto el `/opt/script.php`, y al ejecutarse `/bin/bash` tiene permisos SUID.
Verificamos
![Pasted image 20260115230222.png](../../_assets/57720943ff2fcb3b3593374843a0f636f42072d5f61d98d49188de2201570175.png)

Invocamos una consola
![Pasted image 20260115230301.png](../../_assets/7e359eb4118b8d68fe92d967d4146f91116bb7da23ffcc275808ace660b80a1d.png)
Escalada exitosa a root.

