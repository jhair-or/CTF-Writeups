
**IP**:  172.17.0.2
**S.O:** Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 13 enero 2026
**Autor**:
**Descripción**: 

Enumeración Web
Information Leakage
Enumeration Jenkins
RCE a través de groovy script Jenkins
Enumeración del Sistema de Archivos
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - Path Hijacking 

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260111215924.png](../../_assets/1750da9130cb8ead5cda588816f619b73894d279c269f09dd2401be10389f084.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260111220002.png](../../_assets/41d3333d80d1a9a7810d5ba457d2ca289af84e80f4f88d116db59e955e8d6bf0.png)
Puertos abiertos: 
Puerto 80 (HTTP)
Puerto 443 (HTTPS)
Puerto 8080 (HTTP)

Escaneo de versión a los puertos específicos.
![Pasted image 20260111220506.png](../../_assets/c8154c8e48848845900bf010da877f5bf164093b1c170f63c5723f3dee2cca74.png)


Accedemos al servicio web HTTP del puerto 80
![Pasted image 20260113155034.png](../../_assets/b8248d2777090bc5f3afbf723977315e2f976c4e48cb417b1271dee8bd2161a6.png)


También accedemos al servicio del puerto 8080
![Pasted image 20260111220613.png](../../_assets/23636c02fe038fc267b1f78276f57f91713efa4194e6e15ef013bfea288908a3.png)
Y vemos un formulario donde nos piden unas credenciales.

Buscamos alguna pista en el servicio web del puerto 80
![Pasted image 20260113160721.png](../../_assets/1ea53617f02146806710d726f43ed507e446a1060df69f1ccd6484b78505e6f9.png)

Y al inspeccionar objeto vemos que las imágenes tienen una descripción curiosas como `cassandra`.
![Pasted image 20260113160156.png](../../_assets/7c1dfd9ec3d25eccf48c95c5ed7adb66806ac4eb929d6c0ac6d6408d118eb6b9.png)

Intentamos poner aquellas descripciones como credenciales en el formulario de login del servicio del puerto 8080
![Pasted image 20260113160415.png](../../_assets/05733ad93b84d9b24dcc1271c0a622b697156c1cbd392998dfdeb110c4600ef0.png)
Y logramos logearnos

Al ingresar nos dan un panel, una vez explorado el panel nos dirigimos a el Manage Jenkins
![Pasted image 20260113161521.png](../../_assets/f0ac43a34517821ea90ab65a6228d4aa4e6f202becd8da293cfa0dca4cab6102.png)

Donde vemos que tenemos acceso al tool de Script Console
![Pasted image 20260113161452.png](../../_assets/969104e202a36f0b63b31d1da78b4f51cde3f4f2e355214d280afcec5ec70be8.png)

Y en Script Console tenemos un Jenkins Script Console en groovy.script donde podemos ejecutar scripts en el servidor de Jenkins.
![Pasted image 20260113162036.png](../../_assets/85617e5e11e62b3a41aeb40c757ec520e9fc1ed17fed77e511cde5dff237612e.png)
Entonces generamos una sesión remota.

En revshells.com buscamos el codigo malicioso en `Groovy` que nos dara una Reverse Shell
![Pasted image 20260113163734.png](../../_assets/cd79edf316d212d450804434aba73ae8263e64b767d8eb6989f30d5286729653.png)

Ponemos en escucha el puerto que asignamos, el `4545`
![Pasted image 20260113163749.png](../../_assets/7e21215f838363d16eb22415172ce2fb446f00f7436a763284e8881ab74b316f.png)

Pegamos el código malicioso y lo ejecutamos
![Pasted image 20260113163800.png](../../_assets/910d84b54cf16384cab7451a84b79b9f28c8cdbe341e8c8b5502a11566991fb1.png)

En nuestra terminal recibimos una sesión mediante el puerto `4545` con el usuario `jenkins`
![Pasted image 20260113163820.png](../../_assets/d4551387f19d2185cd8af29ca46a94b037200a09d291aa887ac473bc9762129f.png)
![Pasted image 20260113164000.png](../../_assets/5c7d1748687fc5eabf30d43adc60596aa56ea17464c2d9f52371f3a704492701.png)

Luego de buscar modos de escalar privilegios en la maquina, al enumerar el sistemas de archivos, busco alguna archivo llamado como el nombre de la maquina `JenkHack`
![Pasted image 20260113172137.png](../../_assets/59095ba03fafef847c3469f9518ab06a45db66021da33c120996a63a7b60c5a0.png)
![Pasted image 20260113172210.png](../../_assets/674aea9ec62f1486c597126da3abf113c3f9b7a96fe6d9784e9821626fab3c88.png)
Donde vemos que tiene un archivo .txt y en él hay al parecer un texto cifrado.

Lo llevamos a `dcode` para tratar de encontrar una cadena legible
![Pasted image 20260113172251.png](../../_assets/21c410418551ca0ce673d27a9d0df2fe524387d98f8e3104d17b44d5f6b8fe08.png)
Vemos que puede ser un ASCII85 Econding

Al decodearlo nos da una cadena legible: `jenkinselmejor`, que puede ser una contraseña
![Pasted image 20260113172312.png](../../_assets/f8eff18eca5e444b01f5375a31badb00abf401329b3b186abc99e3eb42dedfae.png)

Al entrar al usuario `jenkhack` colocamos aquella cadena legible.
![Pasted image 20260113172355.png](../../_assets/2328e41ca28d8a72b0008eeb01e549bfd890f8288ebc6be311f906889a47d91a.png)
Y logramos acceder como aquel usuario.

Al estar como el usuario `jenkhack` listamos los permisos sudo.
![Pasted image 20260113172410.png](../../_assets/2efca144e5cc539965a68d6da31a2f60ff69d50e4a6752726338f335a97c3e99.png)
Y vemos que podemos que el binario `bash` esta en la ruta `/usr/local/bin/bash`
lo cual al ejecutarlo notamos que no es un binario, si no un script personalizado.

Nos fijamos en los permisos
![Pasted image 20260113172958.png](../../_assets/40b57eb9516f713884500fbb00c7aedff2a0c1d069fe6f9b3761aa2bde9eb0cb.png)
Vemos que tiene no tenemos permiso de escritura.

Nos fijamos en los permisos de la carpeta
![Pasted image 20260113173359.png](../../_assets/471634abd536dfad241d97e35010e3c53b4ec5abc9aac273cc26105babcc3d0d.png)
La carpeta tampoco tenemos permiso de escritura.

Al intentar leer el binario esto nos muestra
![Pasted image 20260113174522.png](../../_assets/fd61fa315be8ba55466f4792259d1eb56ad69afbc9fd74f884f9e75f32952767.png)

Entonces analizamos el binario con `strings`
```bash
strings /usr/local/bin/bash
```

Y vemos que el binario `/usr/local/bin/bash` 
![Pasted image 20260113174632.png](../../_assets/6b255d73203224ae75e629b45d50942453afd372a37ec80f4b0c5b920f33ee69.png)
`/usr/local/bin/bash` llama a otro script ubicado en `/opt/bash.sh`.

Entonces nos dirigimos a `/opt/bash.sh`
![Pasted image 20260113174919.png](../../_assets/cef0302660571425891cbc187632fc4a86e26c0b35446ce5ea01f3857c3706e8.png)
Y vemos que no  tenemos permisos de escritura.

Y en el directorio
![Pasted image 20260113175045.png](../../_assets/b492910ea7cda41e9fe3a1ae817f1e86f3ee8c60600437679f035faeb8284dee.png)
Si tenemos permiso de edicion, porque el directorio pertenece al grupo **jenkhack**.

Entonces eliminamos aquel scritp dentro de `/opt`.
![Pasted image 20260113175515.png](../../_assets/3de262e0e4f7bfdc394c3ff612abfda6a351982908515b910002c8288ae27b5e.png)

Y creamos uno propio que al ejecutarse nos dará **shell*
![Pasted image 20260113175616.png](../../_assets/74595a00e4c3d38c52c084850f5e10154d764c24e8e713c301b44188b6b3bf3b.png)
![Pasted image 20260113175604.png](../../_assets/22691f21f76ccd80fe3e215c3346456a24858e2a34c1bbbc13d1972dccb96b42.png)

Le damos permisos de ejecución
![Pasted image 20260113175632.png](../../_assets/c6c89c5c51ba15124968132db24b5321c0fdc46459267f884df423f09869ab96.png)

Y ejecutamos el  `/usr/local/bin/bash` con sudo
![Pasted image 20260113175657.png](../../_assets/d6d07dffaea90d2a339f44cacceec4e904473c0d4beff52b4696ee06680a474e.png)
Y conseguimos muna sesión como **root**.