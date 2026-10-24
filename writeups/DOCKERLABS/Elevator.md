
**IP**:  172.17.0.3
**S.O:** Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 20 enero 2026
**Autor**:
**Descripción**: 

Enumeración Web
Fuzzing with gobuster
Bypass de Extensión (.jpg)
Remote Code Execution (RCE) vía Arbitrary File Upload
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] - env, ash, ruby, lua, gcc, sudo

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260120122945.png](../../_assets/cf47bdb0147fd9cf3a96684a8a89f41a38e6f1a2107aaf77a01a1b53ccf1eae3.png)
Hay conexión 
TTL: 64 -> Linux

Realizamos un escaneo de puertos TCP abiertos.
![Pasted image 20260120123031.png](../../_assets/72a4adc34265c37e2ff16d3cf44e18a18cf17afc887882bcab6d4fb373c90723.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo mas a profundo al puertos específico 80. 
![Pasted image 20260120123044.png](../../_assets/6d5c89b2ed2f3940406add3eb8e615f1c7c7785031620bbacc3c7a4b4c293d20.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260120123102.png](../../_assets/0f9d930b6e19262590c58707b1f19c568418c62f21aa41373f66dd8f9be68c1a.png)
Solo logramos ver lo que es un elevador

Hacemos un fuzzing de directorios.
![Pasted image 20260120123647.png](../../_assets/5947602a0b20a069f4a22505ccb3a03054739824b3e0949e0226119d7eee4204.png)

Nos dirigimos a `/themes`
![Pasted image 20260120123757.png](../../_assets/059e2806029e4071650bfe7d4ac3a5d616dac5811283bc8254358d94de6b2e0c.png)
Pero nos dice que no tenemos permiso de acceso.

Entonces hacemos un fuzzing de directorios a `http://172.17.0.2/themes`.
![Pasted image 20260120124229.png](../../_assets/593b7e3a9680f88219c4e311fad7074520b6ae5eec5cd5db10bf27b61a307321.png)
Encontramos `/uploads`

Encontramos un panel de subida de archivos `.jpg`
![Pasted image 20260120123945.png](../../_assets/c4386204d6927194e3887679f5a912f184bbe9dc2e51372682113f45a2a6b22b.png)
Nos indica que solo se puede subir archivos `.jpg`.
Entonces realizaremos un **Bypass de Extensión**

### **Explotación**

Intentamos subir un archivo PHP disfrazado.
![Pasted image 20260120124336.png](../../_assets/7b996ffd8ce842ec1abcf030ce0be8eb717e8ca196768fc39f39ab6805c820fd.png)

![Pasted image 20260120115107.png](../../_assets/ee16a6c02dff89628f2f5f2603d5c18913ce683e66992f81da89551170de705a.png)

Subimos el archivo
![Pasted image 20260120130111.png](../../_assets/b41c638e3b348fbdd15621561447976afdf5c7e88e9d3303ead93552eb504edb.png)

Y nos indica que el archivo se subió correctamente.
![Pasted image 20260120131515.png](../../_assets/d3b38e1ded675f0caa07206edd1cebaadd617c07576394772f286b5313923976.png)

Ponemos el puerto que asignamos en escucha `4545`
![Pasted image 20260120131532.png](../../_assets/c18634c7a4ae206693ef91f9d60d3bcb8e1ffc38ea350368b69dff7495c812ea.png)
Obtenemos una sesión como el usuario `www-data`

### **Post-Explotación**

Listamos los permisos sudo.
![Pasted image 20260120131705.png](../../_assets/b9a54d40a9e00e6956b9ef6856acff07c7ad92b482086533d6df46db75131aa1.png)
Vemos que podemos ejecutar `env` como el usuario `daphne` sin necesidad de una contraseña.

Invocamos un shell con `env` con el usuario `daphne`
![Pasted image 20260120131749.png](../../_assets/2a613354386f63edd17c9aebd558444b4b524e523f5524192432e7fcc2403609.png)
Ahora somos el usuario `daphne`

Listamos lo permisos sudo como el usuario `daphne`
![Pasted image 20260120131823.png](../../_assets/a6d62f2093b1867807436f73fe7081a5453c8e035dee815dfd0788b2d8c2b149.png)
Y vemos que podemos ejecutar `/ash` como el usuario `vilma` sin necesidad de una contraseña.

Invocamos la shell con `ash` con el usuario `vilma`
![Pasted image 20260120131901.png](../../_assets/ec75b8ec83eb5465b22728e87734cbae516abc9a129e4ceb49459ef9fa9fcdc8.png)
Ahora somos el usuario `vilma`

Listamos los permisos sudo
![Pasted image 20260120131935.png](../../_assets/b5a9d2b1b68afb56715fe082df20d28ee7626655b8c5eeabf6f685dcd1b25b05.png)
Vemos que podemos ejecutar `ruby` como el usuario `shaggy` sin necesidad de una contraseña.

Invocamos la shell con `ruby` con el usuario `shagy`.
![Pasted image 20260120132110.png](../../_assets/9d7cfceb669e332d9caedda87416be31faa23448b1904d6f44db3de4fcc483e7.png)
Ahora somos el usuario `shaggy`

Listamos los permisos sudo
![Pasted image 20260120132751.png](../../_assets/e01cf5abde8a9d5110889abda86b09fdcf2ca211880793b4a42c983a9f0f3418.png)
Y vemos que podemos ejecutar `lua` como el usuario `fred` sin necesidad de una contraseña.

Aprovechamos de `lua` para escalar al usuario `fred`
![Pasted image 20260120132229.png](../../_assets/5b80c652bb13f21e2592b35ccfbf07b310c241a4ff6809bf20ff052912018126.png)
Ahora somo el usuario `fred` 

Listamos los permisos sudo
![Pasted image 20260120132330.png](../../_assets/2b3ae219534bc6190fcf817e71c202f2126ae4eaf0085887d9b0e705e00bc9ff.png)
Vemos que podemos ejecutar `gcc` como el usuario `scooby` sin necesidad de una contraseña.

Aprovechamos de `gcc` para escalar al usuario `scooby`
![Pasted image 20260120132320.png](../../_assets/db497bb3a6191a1caf9d522e7bca4a7180caac67422171c3c3856d1e8f8e70a3.png)
Ahora somos el usuario `scooby`

Listamos los permisos sudo
![Pasted image 20260120132432.png](../../_assets/16ffe0ad63733f38af962035ac1d05301c94d1e84e11199f5dbc0cb040ffbbf1.png)
Ahora por fin vemos que podemos escalar a **root**, podemos escalar mediante el binario `sudo` que nos da acceso directo a **root*

Ejecutamos 
![Pasted image 20260120132826.png](../../_assets/d30d170a7de773d05e2738fa3a8f2ad09656288f29d41f8f80036ae816c41f6f.png)
Escalada exitosa a root.