
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 26 diciembre 2025
**Autor**:
**Descripción**: 

SQLI (SQL Injection) 
Information Leakage 
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - env

**Flags**: 


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260124145203.png](../../_assets/08020fc0a995e0d1671e8fb816fb41898f7fe2f364d3f1292f727711d1a3c924.png)
Hay conexión 
TTL: 64 -> Linux

Escaneamos los puertos y los servicios de `172.17.0.2`
![Pasted image 20260113190734.png](../../_assets/7b913c01ff1d3025d4b4a7a929f79c02a9d14aaeb9a88e2877a78465035b164d.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)


Accedemos a la pagina web 
![Pasted image 20260113190811.png](../../_assets/1b76fb88384c27160269cac797653bcb551a35fa85015c971a4b27f9880299de.png)
Y vemos un formulario de login.

Probamos si es vulnerable a SQLi con un payload simple
![Pasted image 20260113190817.png](../../_assets/f6e801eb1319ecc38130787d0212ff4fa716f84c702cef1c51e4ddbfb327a75a.png)

Y si se lograr realizar la consulta sqli y al registrase con éxito nos da un mensaje
![Pasted image 20260113190824.png](../../_assets/e4de38aea519bd97c9a674e0134277335353a41a3af2cf54a0539521da1874d7.png)


Accedemos al servicio SSH con la clave que nos dieron
![Pasted image 20260113190831.png](../../_assets/5ae075e35f916ded98d89d740f0f069c5a6534244c4add8978eba4a370c6eeef.png)
Al listar los permisos sudo vemos que podemos ejecutar el binario `env` como **root** sin necesidad de una contraseña.

En GTFOBins encontramos el comando para lograr escalar de usuario
![Pasted image 20260113190838.png](../../_assets/9d8f153d29e2d17b871751a24ec52597e6f184c323942b83aee5ba78af6f9594.png)

Colocamos el comando 
![Pasted image 20260113190847.png](../../_assets/3a0ebd1e98b6801f40351ef530f8a8f73440371ad18f7bd5812a6b0d0b6f864a.png)
Y logramos escalar a **root**

