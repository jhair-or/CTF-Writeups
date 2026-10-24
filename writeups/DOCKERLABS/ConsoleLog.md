
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 22 Enero 2026
**Autor**:
**Descripción**: 

Web Enumeration
Fuzzing con gobuster
SSH brute force with Hydra
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - nano

**Flags**: 


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260122163153.png](../../_assets/326f3c037825350fb88911f18f33eabee6978759a0cacd086ae920fcc5e19abd.png)

Escaneo de Puertos abiertos TCP
![Pasted image 20260122163125.png](../../_assets/20f7a546ee9ffafe0ee02035f0971ba81f516349fc1fa151ff3580d8ace3240b.png)
Puertos abiertos: 
Puerto 80 (HTTP)
Puerto 3000 (HTTP)
Puerto 5000 (SSH)

Escaneo de versión al puertos específicos
![Pasted image 20260122163210.png](../../_assets/26f097b2d696424dcaf88913dd30700d60c086c61d874172d12fded5f201bcb6.png)

### **Enumeración**

Accedemos al servicio web
![Pasted image 20260122163303.png](../../_assets/0c6568b8105965809e52429a606c537ff8764ca4407ab8ebecb7493d2c85056f.png)
Le picamos al botón.

Y solo nos sale esto
![Pasted image 20260122175731.png](../../_assets/2f7b8bfae333f45460ec5f3848f5a34801731d385d5b4fd8b6b3bc780e44afa2.png)

Realizamos un fuzzing de directorios.
![Pasted image 20260122180733.png](../../_assets/2e8161f58e0d68782970cc5a13e313251ab4de46351f16250d106ab0b9dcb10a.png)

Nos dirigimos a `authentication.js`
![Pasted image 20260122175915.png](../../_assets/0d8b537aafb95b4657a4bef79dd85854a7e384cfe81a4efb35ceff93b6c0962f.png)

Al dirigirnos a `backend` vemos que nos lista una carpeta de servicios 
![Pasted image 20260122180513.png](../../_assets/6ba79aab15ea2190ebcc8b40165697da14f5d4d609e246954f267ae961e9f011.png)

Revisamos el `server.js`
![Pasted image 20260122181007.png](../../_assets/4f456c4e5f8e3a8a3294f86736e18d30ee0e2e01aeccbcffbc637a9f830f306f.png)
Y vemos que dispone de un servicio en el puerto `3000` la maquina y nos brindan una contraseña  que es posible que sea la contraseña de un usuario existente.

### **Explotación**

Entonces procedemos a hacer una fuerza bruta con aquella contraseña y servicio SSH del puerto 5000
![Pasted image 20260122181024.png](../../_assets/7ecbe524e7d6f90a5d62035ea705ff6764f06776bb6edd5c84f6d676650c9075.png)
Encontramos las credenciales.

Accedemos al usuario `lovely`
![Pasted image 20260122181130.png](../../_assets/9194c381f4017fe6e1e32aa05339160bfa91d62ff08ceb28328301a7f00bdd0c.png)

### **Post-Explotación**

Listamos los permisos sudo.
![Pasted image 20260122181142.png](../../_assets/69e4bfdbeb5d55c3732808cfd818daa6171cb040f2f5565a60c253e7739f5054.png)
Y vemos que podemos ejecutar `/nano` como **root** si necesidad de una contraseña.

En GTFOBins encontramos la manera de obtener una shell siguiendo aquellos pasos.
![Pasted image 20260122181240.png](../../_assets/d3f9247aad3876793875d950ff8c6dda7f51caa3bb2b499883dbdf609315659b.png)

Seguimos los pasos indicados y logramos obtener una shell con privilegios de **root**.
![Pasted image 20260122181306.png](../../_assets/b602591b98430b3aa95b37e04eb385288d4c1ba779163f35ba1a1fe000ce5c6f.png)
![Pasted image 20260122181415.png](../../_assets/d368c2a772a203202a537aed0504176e188c98401c09774f1db327570a659e6b.png)
Escalada exitosa a root.


