
**IP**:  192.168.20.138
**S.O**: Windows
**Source:** TheHackerLabs
**Level:** Fácil
**Date:** 8 febrero 2026
**Autor**: 
**Descripción**: 

[[SMB]] Vulnerability - Metasploit

**Flags**: 


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un arp-scan para ver la dirección ip de la maquina.
![Pasted image 20260208151558.png](../../_assets/30ff68d24bdf1e24f08f2a5bb755f09c0395ef67ad8f2ec53e5cab3ea5775aff.png)

Hacemos un ping:
![Pasted image 20260208151621.png](../../_assets/43856cfebdfa0464f2914a607d43b36c4764bce796569bf2d6a7b45955b07382.png)
Hay conexión 
TTL: 128 -> Windows

Escaneo de Puertos TCP abiertos
![Pasted image 20260208151856.png](../../_assets/46666dfdfe955831ba876039917ee4dc5315d274c9938c61b44e25232aca2914.png)
Puertos abiertos: 
Puertos 139 
Puertos 445 (SMB)

Escaneo de las versiones de los puertos específicos
![Pasted image 20260208152153.png](../../_assets/09c9af2e1d66bedee8278f24f370c8d63d60649b3a2eab66e3c7331011812760.png)
Vemos que la versión del smb es una versión que es vulnerable a eternalblue.

### **Enumeración**

Hacemos un escaneo mas a profundo al puerto 445 para ver si es vulnerable a aquella amenaza.
![Pasted image 20260208194208.png](../../_assets/707406052385c5556f783d3cc7f4c4d58bd6f479d66fe4ab6939c44f3fc78600.png)
Y vemos que es vulnerable a **Remote Code Execution (ms17-010)**

![Pasted image 20260208195532.png](../../_assets/9e55f4db47f04e2386c63269b2da923f795b1eab3ff671b91e441ed92a00fb52.png)


### **Explotación**

Abrimos el Metasploit.
![Pasted image 20260208195509.png](../../_assets/fb22ac02c05a7c6693617937b1a0b7595dc308f7c53742a28b8361d532a86af2.png)
Buscamos el exploit para hackear la maquina.

```
search ms17-0.10
```

Usaremos el primero
![Pasted image 20260208195941.png](../../_assets/711b761a7d84d8e467ee9cfe448c17f5fbe9bff3c82f2d3d792979782cf0adad.png)

Vemos opciones requeridas
![Pasted image 20260208195956.png](../../_assets/a3176add1393c544e48b6084fdffc8471aa504c0afaf5bae1dd948ec03eece1a.png)

Le asignamos el RHOST
![Pasted image 20260208200021.png](../../_assets/fe01ad7e9a4a6e4958e4a6eb8dc416d2472a2b7f58d8313614cd33e13bb41af3.png)

Y el asignamos el LHOST
![Pasted image 20260208200227.png](../../_assets/c55414393847571947323dc67540dc8db87d92836683e2f1a6e0f1c40acaaaa2.png)

Ejecutamos
![Pasted image 20260208200333.png](../../_assets/c0b9a4acdc325b3ce131ee28eb6c47f48f2b6afe9bfaed0f66ca718cf21378e6.png)

### **Post-Explotación**

Hacemos un dir y nos dirigimos al directorio `users`.
![Pasted image 20260208200523.png](../../_assets/e218e6b9350370987675cd25f65158afd561c73e8ef2414853b0a323950775ff.png)

Nos dirigimos a `lola`
![Pasted image 20260208200727.png](../../_assets/abb5c00e0cbde640d52c57e7af687a669b30700e532e54bcdb09fcd4e6829ce5.png)
Y obtenemos la flag de usuarios.

Ahora nos dirigimos al directorio `Admin` 
![Pasted image 20260208200803.png](../../_assets/6e12279395d99dbf560468bade893a4200e8534758caed81eced8f1bdde66934.png)
Y obtenemos la flag.
