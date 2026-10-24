
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 23 enero 2026
**Autor**: El Pingüino de Mario
**Descripción**: 

Enumeración Web
Enumeración de Drupal
Explotación Drupal con Metasploit
[[Escalada de Privilegios#**Escalada de Privilegios mediante SUID**#2. Vectores de Explotación SUID]] - find

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260123181712.png](../../_assets/d6cd84f8917762c3d18aa79d9fb36705f0592e15afde8244f88a682e67ad04d4.png)
Hay conexión 
TTL: 64 -> Linux

Realizamos un escaneo de puertos TCP abiertos.
![Pasted image 20260123181747.png](../../_assets/6bdc21195377b0272b9bbcf0be31b1c4d0bd110e3b94dccf2c2a177374be8c84.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo mas a profundo al puertos específico 80. 
![Pasted image 20260123181856.png](../../_assets/3dc0b245be919401cd07c3ea01c47b40de3631095202482098e562e26e2d47e5.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260123181939.png](../../_assets/0177486ac774ab577e1bc1c531191600006d959cc06f56032982086fdcb00d62.png)
Nos muestra un status 403 Forbidden

Hacemos un fuzzing de directorios.
![Pasted image 20260123182921.png](../../_assets/3484e1728afa7f1d126462e194bf96cbc3c5e247fbcaa86deb320aaca9ac1891.png)
Detectamos un directorio `drupal`


No dirigimos a `http://172.17.0.2/drupal`.
![Pasted image 20260123182118.png](../../_assets/0998726257ffc84d47b0767dc4741cee69573f646103308d99d8dced5b5706d3.png)

Investigamos mas sobre las tecnologías y versiones que esta usando la pagina web.
![Pasted image 20260123182339.png](../../_assets/838f89c3f653a851d379f7266515e5215621b136025ae1598e26d639aac9785b.png)
Y vemos que esta usando un Drupal 8

Tras investigar descubrimos que esta versión es es vulnerable a Drupalgeddon2 (CVE-2018–7600)


Buscamos un modulo de exploit en Metasploit
![Pasted image 20260123182604.png](../../_assets/0964cd7be8dbf4ebaca82e0567613950ed26c1dfc1a6e3e437fc83acf5aa71bc.png)
Procederemos a usar el `0`

### **Explotación**

Vemos las opciones que requiere para ejecutar el exploit.
![Pasted image 20260123182758.png](../../_assets/f6f73949a4a95aa5b6936319530de353a9a367a6de9dbeae4245d79cacfed974.png)
Completamos la opción de  `TARGETURI` con `/drupal`.
Y a RHOST con la ip de la maquina


Lanzamos con `run`.
![Pasted image 20260123183246.png](../../_assets/5b884da7fda579ede6e941c3fa25f60316f30a10e731327ca507f0914284f2c5.png)
Y logramos obtener una sesión meterpreter

### **Post-Explotación**

Buscamos binarios con permisos SUID.
![Pasted image 20260123183447.png](../../_assets/2350347b2dccf56d79d030592b86d8e05962d87f5829255af388f3e2888f8e62.png)
Vemos que `/usr/bin/find` dispone de dichos permisos:

 Podemos llegar a obtener una consola como el propietario del binario empleando el siguiente comando:
```bash
find . -exec /bin/sh \; -quit
```


Ejecutamos el comando
![Pasted image 20260123183457.png](../../_assets/b8b9f4c5706ac9e1c1d9b71d30b077cd58bce9b799ea103307244f33793b5e8f.png)
Escalada exitosa a root.


