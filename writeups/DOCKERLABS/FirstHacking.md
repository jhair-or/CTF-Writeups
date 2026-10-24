
**IP**:  172.17.0.2
**S.O:** Linux
**Source:** DockeLabs
**Level:** Muy Fácil
**Date:** 18 diciembre 2025
**Autor**:
**Descripción**: 

Backdoor Vulnerability - CVE-2011-2523

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20251218143136.png](../../_assets/2ae5ec4c1c42db61491e207e2bbcf05c19c89705d79086501112773619a5eed4.png)
TTL=64-> Linux

**Escaneo de Puertos**
![Pasted image 20251218143409.png](../../_assets/ec48f81c25f57a4e173eab3b4d36c1369bea575658e3b3ec18a92a23f61448fc.png)
Puerto abierto:
Puerto 21 (ftp)


Busqueda de exploit para `vsftpd 2.3.4` con searchsploit
![Pasted image 20251218143704.png](../../_assets/f90d353ba1318a8b59cdcb8691096580baa52f9fde2675b9b436df9d4fa534ea.png)
Y vemos que es una versión vulnerable con una backdoor(puerta trasera) conocida.

**CVE-2011-2523**

La vulnerabilidad se activa mediante una "cara feliz" (`:)`) en el nombre de usuario.
Una vez enviado ese usuario, el servicio abre secretamente un nuevo puerto en el servidor: el **puerto TCP 6200**
Si te conectas a ese puerto 6200 (usando netcat, por ejemplo), obtendrás inmediatamente una shell de comandos con privilegios de `root`


Nos conectamos por netcat al puerto 21 colocando de usuario `usuario:)`
![Pasted image 20251218151459.png](../../_assets/19c0fe82a9150181d747e71094bda41227656b50e4212c073936841369de4d96.png)

Ingresamos el usuario y una contraseña cualquiera
![Pasted image 20251218151640.png](../../_assets/515a4d1bec61dedda9fc0c248483dc4ed889433c77e6f6e8f50f57003a87dd59.png)
en este caso 220 nos dice que el servicio esta listo, 331 dice que este acepto el usuario y ahora pide el password.

Entonces cuando cuando recibe "331 Please specify the password" es la señal de que el backdoor ya esta activo y que ya puede conectarse al puerto 6200 y obtener una bind shell

![Pasted image 20251218151737.png](../../_assets/0172e43cb1e2a1104c51a2f55b31172c5498f74648d3336f868551d9e9765036.png)
Escalada exitosa a root
