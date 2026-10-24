
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 3 febrero 2026
**Autor**: El Pingüino de Mario
**Descripción**: 

API Enumeration
fuzzing de parámetros
[[SQLI]] SQL Injection 
Enumeración del Sistema de Archivos
Reutilización de Credenciales

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260204175641.png](../../_assets/bd39bbde055ae1996fdf0abaa91c958b014bcba9dcd23b591bf95ed293d037b7.png)
Hay conexión 
TTL: 64 -> Linux

Realizamos un escaneo de puertos
![Pasted image 20260204175631.png](../../_assets/9451fba4db56dfdd2b89c3323daa6b82f27c0f07e525ae0d351f3378d6fd5362.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 5000 (HTTP)

Nos dirigimos al servicio http por el puerto 5000
![Pasted image 20260204175809.png](../../_assets/06051521c4ad508d34628f298ddc8bf2b537c559991d8f9c7a70f851c46c4656.png)
Y vemos el mensaje:
`message	"No endpoint selected. Please use /add to add a user or /users to query users."`

Vamos a `/add`
![Pasted image 20260204181427.png](../../_assets/63711b24a4460f6d409bbabbc7cd83ce12f8da24323d0fa89fe42259ae7612b0.png)
Y esta bloqueado

Ahora nos dirigimos a `users`
![Pasted image 20260204181359.png](../../_assets/c926b8058eaefeec64903a9b13cd9ce0641f045d1708d666e62c845b177a53f3.png)
Indica que el parámetro es invalido

Hacemos un fuzzing de parámetros con `ffuf`
```bash
ffuf -u 'http://172.17.0.2:5000/users?FUZZ=1' -w /usr/share/wordlists/dirb/common.txt -mc all -fs 35
```
![Pasted image 20260204184647.png](../../_assets/1ae018af10674caf55134a47e176a477b9911ac02a3e54d673ed31c79071d57b.png)
Y encontramos el de `username`

Ahora nos dirigimos al servicio http con el parámetro: `http://172.17.0.2:5000/users?username=1`
![Pasted image 20260204184748.png](../../_assets/16bb7d2b0152813596dec4a4e66d07e6abed0398ca5613288f2d3bca1d86c580.png)
Y nos dice que es un usuario no encontrado.

Probamos con
`http://172.17.0.2:5000/users?username=1'`
Para probar si tiene una vulnerabilidad sqli

Al dirigirnos allí
![Pasted image 20260204184736.png](../../_assets/7af8f5d22d620dc9fc9f2a5f18f41194b140f1b8f8eb6572a85fff7130da3cda.png)
Verificamos que es vulnerable a sqli-
Entonces nos dirigimos con 
`http://172.17.0.2:5000/users?username=' or 1=1 ---`
![Pasted image 20260204184858.png](../../_assets/1b0c29048e476d9742c466ca694dce07ccbd8e524e6a2916acf6d074e30828f2.png)
Y nos da las credenciales del usuario `pingu`


Accedemos con ese usuario al ssh.
![Pasted image 20260204185024.png](../../_assets/9519d16bf0edca2974ef2c37aa8a0ebd6ef20df2ea3b5ae32b216235a72abda9.png)

Hacemos un enumeración del sistema de archivos
![Pasted image 20260204185106.png](../../_assets/ad60c5f10967b8eb39e2c99f69935c103168a47cec698026cdf9eaa82f33b03f.png)

Y en el directorio `/home` vemos un archivo `.pcap` que al proceder leerlo nos da una credenciales
![Pasted image 20260204185120.png](../../_assets/ad0fe36139e00625e8371d0c1a7f2a80e9585b8d916964907fe77ec47d982727.png)

El cual intentamos acceder al usuario **root** con esa contraseña y logramos acceder
Y somos **root**
![Pasted image 20260204185206.png](../../_assets/409782672073235e0bff56540dee87362c66c01d918eed0ba812d2f73ff11ba4.png)

