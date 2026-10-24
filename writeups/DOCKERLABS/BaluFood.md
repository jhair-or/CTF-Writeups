
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 4 febrero 2026
**Autor**:
**Descripción**: 

Enumeración Web
Credenciales Default
Enumeración del Sistema de Archivos
Information Leakage
Reutilización de Credenciales

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260204185940.png](../../_assets/0f8613e25a467e4b130ae8087e0729e11d313c59f43d2f2d06cf4fc72d33244d.png)
Hay conexión 
TTL: 64 -> Linux

Realizamos un escaneo de puertos
![Pasted image 20260204190025.png](../../_assets/6b2f2de212df4335309bd0f519768bf3a39ea100d6545c876cba5001cb88af38.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 5000 (HTTP)

Nos dirigimos al servicio http por el puerto 5000
![Pasted image 20260204190118.png](../../_assets/a97412313a1ed53da8c39009756ab6747ff08a689eb5f7364c2e0fdb14af5a85.png)
Y vemos una pagina web al parecer de un restaurante.

Al investigar sobre que hay en la pagina vemos que hay una parte que dice `admin`.
![Pasted image 20260205162032.png](../../_assets/063d89f762af5ad9c30ae0667e7279a34ae569af09bb08c31e8e044c41aea2a4.png)

Que al ingresar vemos un formulario de login.
![Pasted image 20260205162101.png](../../_assets/961da8dc33b887ca8fddb916da128978a7cf319d794e9cf197a16395e1719bf9.png)

El cual intentamos ingresar con las credenciales `admin:admin`
![Pasted image 20260205162126.png](../../_assets/e87a1b1459f39d89b130d0ec8622e13a906746e8f4ee300dfa29dabe5e75c006.png)

Y al parecer logramos acceder.
![Pasted image 20260205162429.png](../../_assets/8c82abe63ebcef2098e918df4e3d4768c259c1e4e8b7769d0d1d584087bc04e3.png)
Pero luego no vemos nada interesante

Inspeccionamos el código fuente.
![Pasted image 20260205162351.png](../../_assets/76b9417dd6dd01d285717d4ab30a2192e92e72dc13c225c74d3eb35812ff4251.png)
Y vemos un comentario que nos dan unas credenciales de acceso

El cual ingresamos al servicio SSH
![Pasted image 20260205163126.png](../../_assets/ee263a0d52030ab7100c78db3f90711223e23dd1528e7664deb14227322668ff.png)
Y logramos acceder

En el directorio `/home` vemos el usuario `balulero` y `sysadmin`.
![Pasted image 20260205163408.png](../../_assets/0798c1c14bd29762979e573ad30a74e348eb725832755e3710f69b6e04aa8b56.png)

Listamos los archivos del usuario `sysadmin`
![Pasted image 20260205163423.png](../../_assets/4aaab722622c340fff6a0bb5f988746ccf927c79c9bf161f2b7b379819a59fc6.png)
Y vemos un script de Python, así que procedemos a leerlo.

Al leerlo vemos que nos dan una cadena de texto que parece ser una contraseña 
![Pasted image 20260205163458.png](../../_assets/1fb73cbd0f8c0a1b451c1f6edfdd92206ebe96435620adc530956481c2daa71b.png)
Que al probar esa contraseña con el usuario `balulero`  logramos acceder a aquel usuario.

 Ahora como el usuario `balulero` al leer `.bash_history`  vemos que ejecuto un comando llamado `alias` antes de iniciar sesión como **root.**
![Pasted image 20260205163647.png](../../_assets/7af6d66e447d4bc145218462bc6796bf58b71d380cd90f5a3561bc4e47f5765e.png)
Ejecutamos el comando `alias`

Al ejecutar aquel comando nos dan una lista de alias que hay definidos 
que en el alias `ser-root` intenta convertirse en **root** con la contraseña `chocolate2`.
Accedemos a **root** con su e ingresamos aquella contraseña y logramos acceder como **root** 
![Pasted image 20260205163739.png](../../_assets/f9037f4f7a24525b718c97d69697df0a2c36e601b90581f9a573f43bd66c4a32.png)


