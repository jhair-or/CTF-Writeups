***Jhair***
**IP**:  172.17.0.2
**Source:** DockeLabs
**Level:** Muy Fácil
**Date:** 19 diciembre 2025
**Autor**:
**Descripción**: 

Upload File - Abusing Exiftool (RCE)
Fuzzing de Directorios with gobuster
Escalada de Privilegios

**Flags**:  


**Begin**:

---
### **Reconocimiento**:
Ping:
![Pasted image 20251219100630.png](../../_assets/bf11c0274395f2fbc571e242ca22a21501a0304b40d3fa650df791af9138a961.png)
TTL=64-> Linux

**Escaneo de Puertos**
![Pasted image 20251219100848.png](../../_assets/bed467bab148a3939656d486c295db0bb07ce038a5a5e5cbbddf5a32e9349121.png)

![Pasted image 20251219173214.png](../../_assets/e6ef8fb52499a907d55064de084807bf0ec98c673374250e45c946e9d153577b.png)
## **Enumeración:**

Fuzzing de directorios
![Pasted image 20251219144343.png](../../_assets/b8d908586368cbf197bcaf4b4e301601ea90d6140c7a3f4fa91f287f9f4ab267.png)
No encontramos Nada

Descargamos la imagen 
![Pasted image 20251219144801.png](../../_assets/3238746865f03ebce2df0d696d1abcf85aea027a40550823cfb8c4e65fe0c53d.png)

Lo analizamos con exiftool
![Pasted image 20251219144819.png](../../_assets/a1997854e616f5f00660d52d5713d38c9108ccdcfc76108d70677f54676a1157.png)
Nos da una credencial de usuario `borazuwarah`

## **Explotación**

Ataque de Fuerza Bruta SSH
![Pasted image 20251219153302.png](../../_assets/df5d2a3bf3b5c1786c703a00736e71c10230e71b0ad2f255b153549f8798a871.png)
Conseguimos la contraseña de usuario `borazuwarah` que es `123456`

Nos conectamos al SSH
![Pasted image 20251219153827.png](../../_assets/8ec001dfb10cd49bb0639ce8591d2acdb7f793104aaf712b5ae4bdf6a9e45e37.png)
Vemos que tenemos un problema de Verificación de Claves de Host SSH

Lo solucionamos con el siguiente comando:
﻿
```bash
ssh-keygen -f '/home/kali/.ssh/known_hosts' -R '172.17.0.2'
```

![Pasted image 20251219153850.png](../../_assets/a9d83c9352c4cd56d9dc3be6faef21d5d804b813412b64ba1ad85e91c74214f2.png)
y nos conectamos al ssh

Una vez conectados, vemos que tipos de permisos tenemos
![Pasted image 20251219170739.png](../../_assets/ad356a13616a5be94e2ee85efa297f169e15ea5097ff427f0d061575006595cf.png)
primero vemos que tenemos el permiso de ejecutar el comando `sudo` 
y tambien que podemos ejecutar `/bin/bash` con permiso de root sin necesidad de una contraseña.

## **Post-Explotación:**

Ejecutamos `/bin/bash` con sudo para heredar sus permisos

![Pasted image 20251219170827.png](../../_assets/44d5df3255f32ad2cc6140eca563b3de659aa92bed329c9c924f2c5cd6d183a6.png)
Escalada exitosa a root