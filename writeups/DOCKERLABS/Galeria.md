
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 16 Enero 2026
**Autor**: Raul A
**Descripción**:

Web Enumeration
Fuzzing con gobuster
Remote Code Execution (RCE) vía Arbitrary File Upload
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - nano, Path Hijacking

**Flags**: 


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260115231102.png](../../_assets/3a493174add48070c7b4260d4cfbe4a16dd6b1c9c198746755b83c8a1dd7bd43.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260115231204.png](../../_assets/3924521b21d37c084076c7733e7edc75f789c8a3da2c22c889bea20310ce2841.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo de versión al puertos específico 80. 
![Pasted image 20260115231216.png](../../_assets/f5710dbc6ea610e90c66408aef884efefcec2d5aad39fb6aea528101625ade6d.png)

Accedemos a la pagina web 
![Pasted image 20260115231320.png](../../_assets/a57e3b9982f1fc99f26de1019b4a1662813e1a677ce980170279f32004cbc753.png)

Hacemos un fuzzing de directorios.
![Pasted image 20260115231527.png](../../_assets/bba028280ca387bc8b909418affb9523c6594da57e32b56d427d47e0130afeac.png)

Nos dirigimos al directorio `gallery`
![Pasted image 20260115231537.png](../../_assets/25e93dcfa940553347835da757d1f4dcbd5ab45fe7e3285a7f54f2e0ada91fc8.png)
Nos dirigimos a `uploads`

Al irnos a `uploads` vemos que vemos un directorio en php `handler.php`
![Pasted image 20260115231546.png](../../_assets/e91a840f5222ba36b3976f4fac3a224ac490d70242518b0f05be65135691a93b.png)

Al ir a `handler.php` vemos que podemos subir archivos
![Pasted image 20260115231557.png](../../_assets/e3f68d385c38eed4265541dbee7b65ee11d1cb3fb48aae2055e6cd02d36b497c.png)

Creamos un archivo malicioso en php que nos genere una reverse shell a nuestra maquina kali
![Pasted image 20260116132557.png](../../_assets/f18052f208f290fdb470d298cd1cbef51685d4ce05ca7bc3ae2add3d05f8ffd8.png)

Lo subimos 
![Pasted image 20260116132406.png](../../_assets/ea8784a50fe0195266878c68d12bf7b68d69647bafd44a3efb6f42cb226c7fd2.png)

Y vemos que se logra subir correctamente.
![Pasted image 20260116132418.png](../../_assets/2b03a94425945b232e530dd266ad13f9aac0c81a5b9d8c73c91b077793773978.png)

Nos dirigimos a `/uplodas`
![Pasted image 20260116132453.png](../../_assets/ff993ebf98def0ac3c9d0917b4deb15c060298c252f68fff5b875a11bb0f4932.png)
Y lo ejecutamos

En nuestro puerto de escucha logramos obtener una sesion como el usuario `www-data`
![Pasted image 20260116132527.png](../../_assets/d4eced827c02932081bf738f50459ea81f89a465803f0bd099f44042fb87d19b.png)
![Pasted image 20260116132739.png](../../_assets/cea2b9c97b0f9ac6c5a64f9ad69dbe5538710c78d181f399b15f7c742ff49503.png)
A lista los permisos sudo vemos que podemos ejecutar el binario `nano` como el usuario `gallery` sin una contraseña

En GTFOBins encontramos la manera de escalar a partir de `nano`.
![Pasted image 20260116133311.png](../../_assets/f8121e89e87754885fa16a05ab8df0ddb7cd705ceb7b616762f06bd557608672.png)

Seguimos los pasos y ejecutamos el comando
![Pasted image 20260116133519.png](../../_assets/007228f37323d772bcb63ad45c6ed1c4ae4a41ee6220563cfc786632e26bd247.png)
![Pasted image 20260116133626.png](../../_assets/87c3ade11f778769d33a14f723632e91e80093cc99a48bdee1325d445e53dbed.png)

Y logramos escalar como `galley`
![Pasted image 20260116133722.png](../../_assets/f61898cf286bc868070c63c59003dc4d31b8ea983a9b9eb833684366f0ef2239.png)

Listamos los permisos sudo
![Pasted image 20260116133835.png](../../_assets/721872b851644cd643e7f0f570cf2082b14f8439c824a1538be282ba853a907d.png)
Y vemos que podemos ejecutar `runme` con sudo como root sin necesidad de una contraseña
Al ver `runme`, parece ser un ejecutable personalizado

Al leerlo, no nos cadenas legibles
![Pasted image 20260116134617.png](../../_assets/af60f4a9d00aa10670421e527720ad39300a93897a45d6a36a4577e4ccca8857.png)

Lo analizamos con `strings`.
![Pasted image 20260116134949.png](../../_assets/e1905197127b76fe64d9ffcea437fd4b3fa3a5ebdf6ee917b8ded54542a39bb8.png)
Y vemos que aquel binario hace un llamado a `convert`.
El fallo es que el binario está llamando al comando `convert` **sin usar la ruta absoluta** (como `/usr/bin/convert`). Es vulnerable a un ataque de Path Hijacking.
Nos dirigimos al directorio temporal `/temp`
Creamos un archivo falso llamado `convert` que invoque una shell (bash):
```bash
echo '#!/bin/bash' > convert
echo '/bin/bash' >> convert
```
Le damos permisos de ejecución `chmod +x convert`

Cambiamos el `$PATH`
```bash
export PATH=/tmp:$PATH
```

Verificamos el `PATH`
![Pasted image 20260116135757.png](../../_assets/a14b842fa1af3c401edcf92a9e0921911cca59832fdfbfbdeb269e78517267c0.png)

Y ejecutamos el binario `runme` que al ejecutarse hara el llamdo a `convert` quien nos dara una shell como el usuario **root**.
![Pasted image 20260116135859.png](../../_assets/769983525b03845a88ce76ba8a18ae889950f10365bc97f49dcd26b68e2ae1f4.png)
Escalada exitosa a root