
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 20 enero 2026
**Autor**:
**Descripcion**: 

Fuzzing de directorios con gobuster
Information Leakage
SSH Brute Force - Hydra
Enumeración del Sistema de Archivos
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] 
Weak File Permissions-Script Hijacking

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260120120238.png](../../_assets/d2147c71de10b519bffb2aad55b19ae1644faa749a97de86949ee2eb2acf8144.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260120120323.png](../../_assets/111b58d1e022316f1cec3452c0e7b4c9e765fdffe5f5281e8da1987abee30fdb.png)
Puertos abiertos: 
Puerto 80 (HTTP)
Puerto 443 (HTTPS)
Puerto 8080 (HTTP)

Escaneo de versión a los puertos específicos.
![Pasted image 20260120120331.png](../../_assets/3e446ffe3b806de4219d532e48525d521b049f3ea6c52209ba9ac4881da7b93d.png)

Accedemos al servicio web HTTP del puerto 80
![Pasted image 20260120120357.png](../../_assets/90a3aecb70f54d8b86da0cd24c5a541170ec9318cbcfa9b89f2da97fc13ac5c4.png)
El servidor respondió con la página por defecto de Apache2

Realizamos un fuzzing de directorios.
![Pasted image 20260120120804.png](../../_assets/e6e06f2e947d462833301e62ea41df3e63f73bcce0ecae81bb50f841b19a7616.png)

Y al dirigirnos a `/index.php`, aparece una cadena de texto 
![Pasted image 20260120120626.png](../../_assets/35a1848e337c48fab00d08e923be1691643843ebd117b5c4c60d1fc5d6305d39.png)

Con esa cadena de texto probamos fuerza bruta con hydra.
![Pasted image 20260120121024.png](../../_assets/93f9a11ae447e386bd03e5a1c3e1fce3c43c658c0d3c10a9394f70c9adaf1478.png)
Y logramos ver que aquella cadena de texto era una contraseña para el usuario `carlos`

Accedemos con el usuario `carlos`
![Pasted image 20260120121127.png](../../_assets/94d0089f87451e8da8864033550b71ed030b368b2db49a4f7a2695e1e17215f9.png)

Tras enumerar los Sistema de Archivos en el directorio `/opt/` encontramos un script en python que pertenece al grupo root y nosotros somos propietarios
![Pasted image 20260120121631.png](../../_assets/52cee28eddd4073c6d17df62fba0858ad79f0cde2a54fecfdda829e2e1f44683.png)

Entonces nos damos permisos de escritura
![Pasted image 20260120122255.png](../../_assets/c47d8373e20be084d31620ee811c55dde73eab063a0395a012dc9e24cd3790ad.png)

Verificamos los permisos
![Pasted image 20260120122307.png](../../_assets/7b743a41dc26180d5209da3d88c03f27e8d3702fccbf4b9e8800fc052a345068.png)

E imprimimos lo siguiente
![Pasted image 20260120122316.png](../../_assets/08deaa20a90b1644a040633fcbe521479030e12b1aa463c8a5675f7f9e82de7d.png)
Que le dice al sistema que ejecute el comando `/bin/bash`, lo cual abre una nueva terminal interactiva

Ejecutamos el script con sudo
![Pasted image 20260120122339.png](../../_assets/952ce3db1a13e68343c94cf449fb967513d166cd2b5ff1172fca507f987a08a8.png)
Y logramos scalada exitosa a root.
