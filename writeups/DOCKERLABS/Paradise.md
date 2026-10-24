
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 1 enero 2026
**Autor**: kaikoperez
**Descripción**: 

Enumeración Web
[[SMB]] Enumeration smbclient && smbmap && enum4linux
SSH brute forcé Hydra
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] con sed
[[Escalada de Privilegios#**Escalada de Privilegios mediante SUID**#2. Vectores de Explotación SUID]]


**Flags**: 


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260113185432.png](../../_assets/43436a85e3a767fe7a7600fc04554e6db98574d337587d3f241fe3efe1c55d26.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos TCP abiertos
![Pasted image 20260113185442.png](../../_assets/887fd585686a16105c003d2515d146898599bfffbbccaf81e456dd0f3db46815.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)
Puertos 139 
Puertos 445 (SMB)

Escaneo de las versiones de los  puertos específicos
![Pasted image 20260113185452.png](../../_assets/a2ec40ffda77f9f06d29b34322f1e2810190ed510858f3f459e05a724bdd169e.png)


### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260113185500.png](../../_assets/268d2f2dc36ba01a1b0f1d7fe012570e9dde86c59cf902fe9b4f9fa0830b0df6.png)

Enumeración al servicio SMB 
![Pasted image 20260113185507.png](../../_assets/4b80fc2ae23c7cbb554f23fb11d01556137d7633865997711883d33bd6bb1e7f.png)
Vemos dos `Disk` a los cuales no tenemos accesos.

Hacemos un fuzzing de directorios.
![Pasted image 20260113185515.png](../../_assets/1305e6d307d664bada394b9e9a2e1f4b5c2e6630f7627ccbbc41b37f67d84e3c.png)

Accedemos al del `login.php` para analizarla
![Pasted image 20260113185521.png](../../_assets/eae3a195163083672fc28380475778b8391863bd329957a51e1629c5164f40cb.png)


Continuando con la enumeración al servicio SMB
usamos el comando `enum4linux` para obtener alguna información del servicio SMB.
```bash
enum4linux -a 172.17.0.2
```

![Pasted image 20260113185528.png](../../_assets/5f5d3fc9a78fd7178092b3f4557d815facb00eb3104f43666c6607b8e553bafb.png)
Logramos ver dos usuarios: **andy** y **lucas**

### **Explotación**

Hacemos fuerza bruta primero con el usuario **lucas**
![Pasted image 20260113185537.png](../../_assets/b6a8869768c3bdfe3a19b7b3d6d6b13922e27647a8a98db97bc3e0505cde7dac.png)
y logramos obtener las credenciales.

Accedemos al usuario **lucas**
![Pasted image 20260113185557.png](../../_assets/d06679ab371115a97ce6b519ca992b5017bb516c42053985380ab872aa4f9973.png)

### **Post-Explotación**

Revisamos los permisos sudo
![Pasted image 20260113185603.png](../../_assets/00844bf64c047cdf39da61423e6667af30ffba37e5005ef810e226acaee27f46.png)
Vemos que podemos utilizar `/bin/sed` con sudo como el usuario **andy** sin necesidad de una contraseña.

Vemos que podemos invocar una `shell`  
![Pasted image 20260208164859.png](../../_assets/91c66d427b82b4dee5f5e66cad27c0c9801b6c3664532924368ba45494e84280.png)
![Pasted image 20260113185610.png](../../_assets/ea5813d7804b926fb501c84e04ad493e5d84d82a6d7eb32302766f9b91cf5a5d.png)
y logramos escalar al usuario **andy**


Buscamos binarios con permisos SUID.
![Pasted image 20260113185617.png](../../_assets/5b70b280a9e1dd17a8ed0c917f3333690df13b29e277753d4ce6b57be3102077.png)
Vemos uno peculiar llamado `privileged_exec`
Que al ejecutarlo nos da privilegios de root.

![Pasted image 20260113185623.png](../../_assets/9a76dd02ebb2ec71b8627ab93982cfbee026889594691afcb926ddc82a533e73.png)
Escalada exitosa a root.