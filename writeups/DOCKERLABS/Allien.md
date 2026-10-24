
**IP**:  172.17.0.3
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 1 enero 2026
**Autor**: Luisillo_o
**Descripción**: 

Enumeración Web
[[SMB]] Enumeration smbclient && smbmap && rpcclient && enum4linux
SMB brute force with nxc 
Remote Code Execution (RCE) vía File Upload
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] con service

**Flags**: 


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260208165935.png](../../_assets/c746f562a9c70cfc46cfeed7fd28755a75401de209b7dc4406be23aa0ff91f22.png)
Hay conexión 
TTL: 64 -> Linux

Realizamos un escaneo de puertos
![Pasted image 20260208165921.png](../../_assets/5d108c6da0ce68b01cc592b7929dae1b6ab7abad52c26bb0f31cadbe2be6b662.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)
Puertos 139 
Puertos 445 (SMB)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260208170000.png](../../_assets/6780ebc986727e89ea2bee76a7b42c4f05c515d02e3697543cd16a4d4d55e666.png)
Nos muestra un formulario de login simple.

Hacemos un fuzzing de directorios.
![Pasted image 20260208170015.png](../../_assets/13e7523196a9abfc6a185e9eb8fafb6a8ce896c202e9ddadd495d92c8c07e932.png)

Nos dirigimos al de `productos.php`
![Pasted image 20260208170024.png](../../_assets/80222b97cbc755686f44fd657225c303a75e944daa4fd4329e18d7013c7f1598.png)
Pero no encontramos nada en ningún directorio al final

Procedemos a enumerar el servicio `smb`.

Primero listamos de forma anónima los recursos compartidos
```bash
smbclient -L //172.17.0.3 -N
```
![Pasted image 20260208170035.png](../../_assets/70e56ed6d06b99a305a8ce35c48889e0d86b66ec2df2a8796b8b32faf1049182.png)
Vemos que hay un `Disk` que dice **Carpeta compartida sin restricciones**.

Hacemos una enumeración de permisos sobre los recursos compartidos.
```bash
smbmap -H 172.17.0.3
```
![Pasted image 20260208170049.png](../../_assets/f019a0b20936590c1fb876ac58816f22e8d9e373d2883575598317d9f3d00552.png)
![Pasted image 20260208170100.png](../../_assets/2a5113bd060b1a23a406a74e0c11395e4ed69eb089c0dee585bcf3b81d8b2760.png)
Vemos que para el `Disk` tenemos el permiso de **Read**.


Interactuamos directamente con `myshare`
```bash
smbclient //172.17.0.3/myshare -N
```
El **`-N`**: Indica "No Password"
![Pasted image 20260208170131.png](../../_assets/54411a622e26797149d9dd6af28c998757c45e74b66780de96a0dcf766da8ede.png)
Vemos un archivo `access.txt`
Lo descargamos con el comando `get`

Al realizarle un `cat` vemos una cadena larga de texto que parece ser un token JWT
![Pasted image 20260208170140.png](../../_assets/36f2f9c88d74481ab36d14287ec364fe57e9c5780cc624ba5c549e68865725b3.png)

Procedemos a hacer una enumeración de usuarios

```bash
enum4linux 172.17.0.2
```
![Pasted image 20260208170211.png](../../_assets/52d6ff8198d24af618a76f0bef2cd8c7e5d5382d7684aa38badd7938bd3670f8.png)
![Pasted image 20260208170220.png](../../_assets/d8f47c5897182f1e15b590a8a8d53321bd8ea0623540bf7e207998efcb6755aa.png)
Vemos los usuarios **satrian7**, etc.

O también podemos enumerar los usuarios con `rpcclient` .
![Pasted image 20260208170231.png](../../_assets/7e408e25561bb0cf2d8ce02ee480602a63c3afac2420c8f4d6b433c86aa948a7.png)
![Pasted image 20260208170247.png](../../_assets/b21befb8a0c86b1bf3943b536a05956ff2db3bf257eb7b878e5f060942885d87.png)

Hacemos un ataque de fuerza bruta con `nxc` para los usuarios encontrados
```bash
nxc smb 172.17.0.3 -u users.txt -p /usr/share/wordlists/rockyou.txt --ignore-pw-decoding | grep "+" --line-buffered
```
![Pasted image 20260208170318.png](../../_assets/39a89362246cf909b35cdf18ccb04f9e4e276c76a6749ae7698e1c8921267f4f.png)
Encontramos las credenciales del usuario **satrian7**

Procederemos a ver qué permisos tenemos con el usuario **satriani7**.
```bash
smbmap -H 172.17.0.3 -u satriani7 -p 50cent
```
![Pasted image 20260208170351.png](../../_assets/1bb11f677b92365c4c9818f6542f80ef9a2f7832df64176fb14981af0c473290.png)
![Pasted image 20260208170359.png](../../_assets/ca09c0bf59daa60cc62bdea6898ba26c68833ba0850bb9cdd9b8902152290e07.png)
Vemos que podemos leer el `Disk` `backup24`

Abrimos una sesión interactiva en la carpeta compartida `backup24`.
```bash
smbclient //172.17.0.3/backup24 -U satrian7
```
![Pasted image 20260208170406.png](../../_assets/1d4da2532331e9b59c8bd36de14f90d9f89ea179be76f90919436bf21e332180.png)
Vemos varios carpetas.

Nos dirigimos a `Documents`
![Pasted image 20260208170414.png](../../_assets/d33195890038f8ca1497bce12f35dc392e2484b38b87d1fc03911d2c1a8083f7.png)
Vemos que hay dos directorios: `Personal` y `Work`
En el de `Personal` encontramos dos archivos llamados `notes.txt` y `credentiales.txt`

Lo descargamos.
![Pasted image 20260208170424.png](../../_assets/df3922063bd3c00d5907708f1ef0d472abd530a4276947a39afaecc8713e5baf.png)

Dentro de `notes.txt` dice:
![Pasted image 20260208170432.png](../../_assets/c59de1225604cbedd6180709d6f1edf544f41cbceb81f103253cd0deee48a748.png)

Y dentro de `credentials.txt` dice:
![Pasted image 20260208170450.png](../../_assets/8553915b5128f1898d2b15dfa9e7cbd6b0d8027a5b11a6c609792757aeb01ca8.png)
Vemos una lista de credenciales de varios usuarios.

Vemos qué permisos tiene el usuario `administrador`
```bash
smbmap -H 172.17.0.3 -u administrador -p Adm1nP4ss2024
```
![Pasted image 20260208170505.png](../../_assets/8896c28f82f9e4680374b340419b074318562523c9c0f50a2a48becb5b49326f.png)
![Pasted image 20260208170513.png](../../_assets/9ca480dae2ce9c7689b64599de71710729eb4e8124b6f8fe8c02b5584e5324af.png)
Vemos que tenemos permiso de escritura en `home`.

### **Explotación**

Nos dirigimos a `/home`.
![Pasted image 20260208170523.png](../../_assets/69078ba2979fe855fc1ae5683ffddc7e9985254495c259ac3faac7f627eac152.png)

Creamos un archivo `.php` para establecer `shel`l interactiva
![Pasted image 20260208170529.png](../../_assets/66e1c91b2bfbabefaeb8edd5980f5cd24db2ad8b2ee213ed7b4fa04b0685a43d.png)

Lo sacamos de https://www.revshells.com/
![Pasted image 20260208170537.png](../../_assets/109987b37e19d93957e72722922ab7814f2932286516f51c321ce0d625ecee7b.png)


![Pasted image 20260208170545.png](../../_assets/2a87f0a3bca5c903f903b1cf47432d8cc1d9d51709fa40d05913bfe6eaa2d4ff.png)

Lo subimos a `home`
```
put reverse.php
```

Luego nos dirigimos a:
`http://172.17.0.2/reverse.php`
![Pasted image 20260208170646.png](../../_assets/02943cf08bd79f22599ae4e7aaf033310d31f6cad325efd3c75b63dd53131b1f.png)
Ahora lo siguiente seria establecer una Reverse Shell

Ponemos un puerto en escucha
```bash
nc -nlvp 4444
```

Establecemos una Reverse Shell
![Pasted image 20260208170655.png](../../_assets/2719d5a277ec9b443b082b449ba4fe1312583be58895b321766751adf9667f1d.png)

Y logramos obtener la **shell** en nuestra terminal
![Pasted image 20260208170704.png](../../_assets/0cb8010f5ade9d55e50a661f9f1f06bfd9d3d2913de4aeb12a5ebce919badbbf.png)

### **Post-Explotación**

Revisamos los permisos sudo
![Pasted image 20260208170712.png](../../_assets/a598a35b0c7b39869cb5390153fa0cdfaa2b6fcf22008447ff94edabb07a2828.png)
 vemos que podemos ejecutar `/service` como **root** sin necesidad de una contraseña.

En gtfobins encontramos como escalar de privilegio a partir de `service`.
![Pasted image 20260208182511.png](../../_assets/229de960604deedf7f9e12a705d124ab5ce49bbbab2eaf4e88ad555cd1416671.png)

Ejecutamos y logramos ser **root**
![Pasted image 20260208170725.png](../../_assets/01a38511be85d26bed6dfaa7765880ad57f768fb3c1fd7b4ab6d4bd4c81db6a5.png)

