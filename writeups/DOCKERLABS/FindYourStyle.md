
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 19 enero 2026
**Autor**: El Pingüino de Mario
**Descripción**: 

Enumeración Web
Enumeración de Drupal
Explotación Drupal con Metasploit
LinPeas Recon - Enumeration
Information Leakage
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] 

**Flags**: 


**Begin**:

---

### **Reconocimiento**:

Comenzamos haciendo un ping:
![Pasted image 20260119124114.png](../../_assets/ef2a8a8400a71917bd97e970ec933b9f10a423bc5a19ab6ca8277ec263511013.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260119124149.png](../../_assets/ae08c003c85ecb643d7fb5a7483b86ae4114574f63b27fa0cfb5c27d4cd3f9cc.png)
Puertos abiertos: 
Puertos 80 (HTTP)

Escaneo de versión al puertos específico 80. 
![Pasted image 20260119124202.png](../../_assets/a3e1d42f3c1c5df2ad460f4a30f07f0eb0f2035eb4d8e09bf2e6799c33c235a6.png)

### **Enumeración**

Accedemos a la pagina web
![Pasted image 20260119124226.png](../../_assets/1cb3cdddddbaef5323f39aa0a3da4d4dc06ed67ccb1d42a2c8c8ddf270e48500.png)
Vemos que estamos ante un CMS **Drupal**

Utilizamos **whatweb** para ver mas a detalle las tecnologías que utiliza.
![Pasted image 20260119131611.png](../../_assets/6dcfc7763f6104e24eee7f1e5278a86b5483ebe9d5bce1e484385fcde086c5a9.png)
Vemos la version del Drupal

La buscamos en [[Metasploit]]
![Pasted image 20260119131946.png](../../_assets/80ba28ccc73df8c93c970815f7d3597dd84284a293c3f1d2dca33c9ec02e6291.png)

### **Explotación**

Y usamos el `exploit 0`.
![Pasted image 20260119131955.png](../../_assets/29bc034c7135a1bd8b696506e8067438b633eb023cef5d98b214662a77aa82d1.png)

Vemos las opciones que requiere para ejecutar el exploit.
![Pasted image 20260119132023.png](../../_assets/ee948e87dba33552d3276046028cc5cce03182980827d3ec30719b61aaee1114.png)

Le asignamos **RHOST**
![Pasted image 20260119132152.png](../../_assets/21b094b730dfe33c21d91173c95b474a4de1bc1c5c795e3d5f3d2d1de7ddeb84.png)

Una vez completado todo lo lanzamos con **run**
![Pasted image 20260119132354.png](../../_assets/4aaffa188ffd1a5f47d7a377840d0c563d261f9e174d88232b89f4338d683cb4.png)

Obtenemos la RCE
![Pasted image 20260119132641.png](../../_assets/a07a2effadedcae81bf06fbca26a0718215b9f51fcbc9bd9f046d4d37306785a.png)
Vemos que hay el user **ballenita**

### **Post-Explotación**

Después de varias enumeraciones para poder escalar privilegios, decido utilizar `linpeas`.
Montamos un servidor web rápido con Python
![Pasted image 20260119133332.png](../../_assets/4d9835a2fc54d6f2cb8c3bf490c9ac380f2c64325b2e8afc0cdae37cd3b836f9.png)

Desde la maquina victima descargamos el archivo
![Pasted image 20260119133615.png](../../_assets/0c718ab7a236bb0565b394e9062b48123e3e5f5bfe463b181dc8e9bd839bc495.png)

Le damos permisos de ejecución
```bash
www-data@7c43d7438a5f:/tmp$ chmod +x linpeas.sh
```

Ejecutamos
```bash
./linpeas.sh
```

![Pasted image 20260119134352.png](../../_assets/741eb9953bef96ee185fe56fb3ef4875f66086b9661095f90483b2c1d876ccd9.png)

Vemos que encontró unas credenciales del usuario **ballenita**
![Pasted image 20260119134307.png](../../_assets/0d5f644632c72719ccf96611a0332092748e4757600c7918c92d84cdfbfe4cb9.png)

Accedemos a **ballenita** con esas credenciales
![Pasted image 20260119134453.png](../../_assets/89acf343cf83b9619c4ff69e902cd35ac159845319536caf8bc08440dcb4fd2e.png)
y vemos que logramos acceder

Revisamos los permisos sudo
![Pasted image 20260119134503.png](../../_assets/04d24c68f7e6182908730ab10d4db2cc72059dfbd0b94675a4104ebcbb7f032c.png)
Vemos que podemos utilizar `grep` y `ls` con sudo sin necesidad de una contraseña.

Utilizamos `ls` para listar el directorio **root** y ver si hay archivos 
![Pasted image 20260119134738.png](../../_assets/cf74cc41331542bc62614e037c616ef76c17ff4d0ca287d3b010c520bdbc7156.png)
Vemos el archivo **secretitomaximo.txt**

Con `grep` leemos el archivo .txt
![Pasted image 20260119134827.png](../../_assets/0c3e92c20a08fce6934db402c1f78da60f7e7a312a35c6b0bd87cac91420b26e.png)
Nos dan al parecer una contraseña

Accedemos a **root** con esa contraseña
![Pasted image 20260119134854.png](../../_assets/2ac6ffd77c72ca8a974fa0f6371bb7f06919813ce52ba00132f4c2fc38371191.png)
Y logramos escalar exitosamente a root.

