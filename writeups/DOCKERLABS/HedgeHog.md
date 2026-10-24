
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Muy Fácil
**Date:** 23 diciembre 2025
**Autor**:
**Descripción**: 

Reverse Dictionary Attack 
SSH brute force with Hydra 
[[Escalada de Privilegios#**Escalada de Privilegios mediante Sudo**]] 

**Flags**: 


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260113191120.png](../../_assets/1823dbdd36e4b67a14497dbb5209dab4a7d6bac219bc00dbc188913b5ec55046.png)
Hay conexión 
TTL: 64 -> Linux

Escaneamos los puertos
![Pasted image 20260113191127.png](../../_assets/2be43e168df4bbb5abd56c490179f0d548ed23562ce1724b5909b56fbc191dbe.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

Accedemos a la pagina web
![Pasted image 20260113191135.png](../../_assets/2643d0f4e19eefa311ff9623a8943b8e2c007a0ea63a927a65dc5a65648d9920.png)
Y solo nos muestra la palabra `tails`

Después de enumerar e buscar varias cosas
Se opta por realizar la fuerza bruta con `tails` como usuario y con el diccionario `rockyou.txt`
![Pasted image 20260113191147.png](../../_assets/3b47d40eaa6f2039bd348e4af610fb621226f56f242e5cfbde953d6a3b7e7f46.png)
Como no se llego a nada

Lo que luego se realizo fue dada la palabra `tails` se revolvemos el diccionario `rockyou.txt`
```bash
tac /usr/share/wordlists/rockyou.txt > ~/rockyou_reverse.txt
```

Eliminamos lo espacios en blanco
```bash
sed -i 's/^[[:space:]]*//' ~/rockyou_reverse.txt
```

Y procedemos a realizar la fuerza bruta con el nuevo diccionario revuelto de `rockyou.txt` 
![Pasted image 20260113191226.png](../../_assets/ce2eb4c5e41111340e372a30413dcf4ae873dffe19a6bb370c0d60b7476ec1fb.png)
Lograos encontrar la contraseña validad para el usuario `tails`

Accedemos al servicio SSH
![Pasted image 20260113191234.png](../../_assets/02f9550cf9e5becb6ff6a15d482c6077a255bc105c8a24632ee02ef56f4559d7.png)

Enumeramos el sistema
![Pasted image 20260113191251.png](../../_assets/8476f9e933fd8f008af9641621a842b10b6855dc82017e1a3e9102994d16ab93.png)

Y al listar los permisos sudo, vemos que podemos acceder como el usuario `sonic`
![Pasted image 20260113191258.png](../../_assets/bb02917700792483549c0abe2aabb27c96765f8ed152c45115954a97636d91ed.png)

Accedemos como el usuario `sonic`
![Pasted image 20260113191313.png](../../_assets/a1a10ac1e93301ad6901357c0dbecec8985f39e1e50afe02e95100bd6d533990.png)
Y al listar los permisos sudo del usuario `sonic` vemos que podemos ejecutar cualquier comando como **root** sin contraseña

Accedemos al usuario **root** directamente.
![Pasted image 20260113191319.png](../../_assets/8faef5b63715de863517d887040d97eb07a7516bd718d014747343c8a475b25f.png)

