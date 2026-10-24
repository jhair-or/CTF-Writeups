
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 1 febrero 2026
**Autor**: El Pingüino de Mario
**Descripción**: 

Enumeración Web
Vulnerabilidad IDOR
[[Escalada de Privilegios]]
Information Leakage

**Flags**:  


**Begin**:

---
### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260205164728.png](../../_assets/c7401939c912927024c679f20dd203014c3840bb021909e0e00fe6222552ac0a.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos 
![Pasted image 20260205164802.png](../../_assets/5c8070d13723ef8cf9c308b72342ecec919d9a7a57a529bee88f9d0794106787.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260205164830.png](../../_assets/eadcf0fe7404387ac8585876bd25c7d566a7cc50f562398fd82e9f28f09d614a.png)
Y vemos un formulario de login.

Nos creamos una cuenta y accedemos a ella
![Pasted image 20260205165210.png](../../_assets/ef33619e38f0840ef5b79107cc3c6a25ef552b9417777915fe85810a0e69dc61.png)

Al acceder vemos que dice `Bienvenido, user1` y en la parte del link vemos que nos dan un `id=55`
![Pasted image 20260205165318.png](../../_assets/61e0f228b7bb8742eeaddfb557ae6f6779a91bfcbb245f1efa67c89fa386d133.png)

Viendo mas la pagina, tambien vemos que nos muestran nuestra contraseña hasheada
![Pasted image 20260205165755.png](../../_assets/b593974eebbefcab6507ecf7fd2f8634c976536ad3f6e391176dbb4179c380e3.png)

Tras varios enumeraciones vemos que es una
Vulnerabilidad IDOR

Como vimos al principio al acceder vemos que el link que nos dieron el `id=55`
![Pasted image 20260205165806.png](../../_assets/07babb4f9454a1072254d5f2918e5967cf08b71ab7845ebb0049bbf6a7a34aa9.png)

Y empezamos a cambiarlo y probar con el `1, 2, 3` y que con el `3` se logro acceder al usuario `juan.perez`.
![Pasted image 20260205170203.png](../../_assets/fd9b052a00d902f6af7d63c2034b8fa2ce9997940c6dba772c3add3869f82922.png)

Después de varios pruebas, se logro acceder al usuario `aidor` que es el mismo nombre de la maquina.
![Pasted image 20260205170250.png](../../_assets/fc107e5cb720b93fa45b4239f2048bbd20cbf2546d48cb7ad28a4774b26cc539.png)

Vemos su contraseña-
![Pasted image 20260205170356.png](../../_assets/3153076c3dacc1950d9d664fdd5b39402fc8570374dd9debbad27b8051aaf23b.png)

Lo dehasheamos.
![Pasted image 20260205170404.png](../../_assets/c1032fc0a8dd57e3b6499b9244c112ed5a258369681f60723d0e3f59dffad5db.png)
Logramos obtener su contraseña `chocolate`

Accedemos por SSH
![Pasted image 20260205170454.png](../../_assets/5995169215cd35dc67cdf12867026e501e773fb37996665673d416c39132b9ee.png)

Enumeramos del sistema de archivos y en el directorio `/home` vemos un archivo en Python.
![Pasted image 20260205170941.png](../../_assets/60fde913dfd5688d7ac8a3a0a2a6169515122c4e47b82c4d16b54ac70baba088.png)

Que al leer se descubre la contraseña de **root** hasheada
![Pasted image 20260205170925.png](../../_assets/a7eda720393a1c79783371becbb08b8ae83d5f15fca5f5c9578275e2e724bfba.png)

Utilizamos una herramienta de cracking de hashes para obtener la contraseña en texto plano.
![Pasted image 20260205171641.png](../../_assets/fcf4403eecf53326a08d865a8f52112a55a60a14477fdf7185698bdd0d866e7b.png)
Obtenemos la contraseña: `estrella`.

Y logramos acceder como el usuario **root**
![Pasted image 20260205171703.png](../../_assets/1c1ceb732546ad0cf642765833c75af5f3ed87c9e98ed9ac1270573fb8af6c4c.png)




