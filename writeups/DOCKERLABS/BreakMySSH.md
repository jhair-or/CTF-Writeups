
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Muy Fácil
**Date:** 19 diciembre 2025
**Autor**:
**Descripción**: 

SSH brute force with Hydra

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20251219220151.png](../../_assets/283b18c4008d1287ecbff96ff39467221cc1aac03882c63d3d2c7c4714af96c1.png)
Hay conexión 
TTL: 64 -> Linux

**Escaneo de Puertos**
![Pasted image 20251219222451.png](../../_assets/b95237a0ed86e32fa7f93337f1d3d65a8be58a8f1ec26add1b18a57741813dd8.png)
Puerto 22 abierto con el servicio de ssh


Como vemos que solo hay el puerto 22 realizamos una fuerza bruta con hydra al servicio SSH.
Hacemos un ataque de fuerza bruta de usuario y contraseña
![Pasted image 20251219225043.png](../../_assets/663cb653f583cb454aa2bb340e72efe966b9f069f0cf86a6b32a0e816fc88d4b.png)
Logramos encontrar que existe el usuario `root` con la contraseña `estrella`.

Y accedemos como el usuario **root**
![Pasted image 20251219225938.png](../../_assets/07a92b6a4c4cf0890c40bd21564db580dbd3a34cc27ed7f061f3cd5a2513b956.png)
