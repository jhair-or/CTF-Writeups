
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 13 enero 2026
**Autor**: The Hackers Labs
**Descripción**: 

Enumeración Web
Criptografía
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] a través de un Shell Escape

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260110165415.png](../../_assets/564cabfccc629484e627b387f95085b93f19f3b6cdf48c0d9fddbe084525e0cb.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260110165632.png](../../_assets/af3ad632c40f16dce299f7387957b7a8b4bac67987cb9e59a8ea9afb58629d25.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

Escaneo de versión al puertos específicos.
![Pasted image 20260110165615.png](../../_assets/37822bd1bc033c375975353790cca503b7bb2a63c2b2048326059023ba952c35.png)

Accedemos a la pagina web
![Pasted image 20260110165723.png](../../_assets/58f32886301201cb72602830e956a37ee85150a3436f54be00ce2b7d2038494c.png)
El servidor respondió con la página por defecto de Apache2 en Debian

Realizamos un fuzzing de directorios.
![Pasted image 20260110170024.png](../../_assets/1ce1b19d2d71aaecfcb93d57105c959b650532f7d5a09ca8909315a68b608d49.png)

Nos dirigimos a `/images`
![Pasted image 20260110170052.png](../../_assets/7acfa2784f26a2e18d1da3e458835eebbef2b42f8f3a42409f9999eb63cb0e02.png)

Descargamos el `agua_ssh.jpg`
```bash
wget -r http://172.17.0.2/images/agua_ssh.jpg
```

Lo analizamos los metadatos con `exiftool`
![Pasted image 20260110170241.png](../../_assets/85076ae9859f07f358c792691d05219973fd4fc818e04b974750bdadab92e727.png)
No encontramos nada

Intentamos extraer información oculta por esteganografía pero no encontramos nada.
![Pasted image 20260110170859.png](../../_assets/0210d151d8e6c55963ed1df8f4544ab7e5f1a9e7b0e0104fa1afaf6441d60b9e.png)

Nos fijamos en el codigo fuente de la pagina web
```bash
curl http://172.17.0.2
```

Y vemos esto
![Pasted image 20260110172150.png](../../_assets/edff51fd22d83456d41435001b95ce182d3cb0ebd4fcd4024d66054e6b1974f2.png)

Que lo llevamos a `Dcode` para analizarlo
![Pasted image 20260110172214.png](../../_assets/1dcb83d90256f14e62bcfadc1673a5b3968a3b366b88f58fe819ede5d5d6c6ac.png)

Y lo logramos decodificar
![Pasted image 20260110172234.png](../../_assets/ee04784072a3df17c4c80cd92aa7aa30ec7920dc33e2e277da0104ebd16c5487.png)
Nos da una palabra: `bebeaguaqueessano`

Teniendo en cuenta que había una imagen que se llamaba `agua_ssh`
Intentamos ingresar con las credenciales de `agua:bebeaguaqueessano`.


```bash
ssh agua@172.17.0.2
```


Revisamos los permisos sudo
![Pasted image 20260110181245.png](../../_assets/ca822890753f65c840321d2aeddc3528ec1bcb0a5244c66d60e88ed48d62dc3c.png)
Y vemos una herramienta peculiar llamada `bettercap`


Analizamos a aquella herramienta
![Pasted image 20260110180927.png](../../_assets/a2a91ac1b135ad29ec78b74835126a1dd6c869cbbb91a4586510265d5b56eae1.png)
`!` permite ejecutar comandos directamente en la terminal del sistema operativo sin salir del programa.
Entonces ejecutaremos un `! /bin/bash` o le damos permisos suid a la bash.

![Pasted image 20260110181154.png](../../_assets/bde02adc32c894e3cb5b3fd28879c121d33101c70e62ae7ca8036ee157044286.png)
Le damos permiso de ejecucion a /bin/bash y heredara el permiso de root.

Salimos 
![Pasted image 20260110181355.png](../../_assets/343f228323e2fdcd51ab24bbaa878b83b68c624129b596802ff128f9f312cde6.png)

Y ejecutamos /bin/bash
![Pasted image 20260110181430.png](../../_assets/c949063f803d7d597cd38ef642fa48f3aba13b36829345c6cb1f8d74df3f6f75.png)

Y nos dan una shell con permisos de **root**
![Pasted image 20260110181459.png](../../_assets/3f55388d112bb6986da3c3e50f16696a4deeb389253d90e6d5cc179529c0d9aa.png)
Escalada exitosa a root