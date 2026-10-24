
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 30 enero 2026
**Autor**:
**Descripción**: 

Web Enumeration
HTTP Basic Authentication Fuerza bruta con Hydra
Ataque de Diccionario de Credenciales Combinadas
Fuzzing con gobuster
Information Leakage
SSH fuerza bruta con Hydra
Transferencia de herramientas con SCP
[[Escalada de Privilegios]] mediante fuerza bruta con un script de github


**Flags**: 


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260130130144.png](../../_assets/47c5b9cea8bdcd7c840f78b7b3359d7c88c288c57f31eea6d6a2283d4869f642.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260130130125.png](../../_assets/2e889166f1c17115af7e5a9837a2a8910ce4f1c0dd32d3d5efe8d89dcc3ebbfa.png)
Puertos abiertos: 
Puertos 22 (SSH)
Puertos 80 (HTTP)

Escaneo de versión a los puertos específico 22,80.
![Pasted image 20260130130234.png](../../_assets/abe86e4da31dd27bf82c83a1996af0920f2f0c03c8019c35c5afc7a4ec3e737f.png)


Al acceder a la pagina web nos piden un usuario y una contraseña
![Pasted image 20260130130224.png](../../_assets/b17616abed21ce41edf4f9f9335ecd0478ac44771719aba449b5ec639325dda2.png)

Interceptamos la petición con `burpsuite`
![Pasted image 20260130141053.png](../../_assets/d1cba23980552c87b99c8c17c84b5558442f7c8549db6fa477388ff0e5f4d20a.png)
Vemos que realiza una petición `GET`

Entonces lo que hicimos fue realizar fuerza bruta con hydra
```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt http-get://172.17.0.2
```
![Pasted image 20260130132046.png](../../_assets/f2e7162d77c885e60915ae1f32d791074cd28e12effad796abfac9191fb86fad.png)
Y logramos encontrar las credenciales de acceso a la pagina web.

Una vez que logramos acceder ingresando las credenciales, las paginas nos respondio con una página por defecto de Apache2
![Pasted image 20260130141529.png](../../_assets/282efc40200d49d51a8b1cc2c3543c63274cdb48156c8390d3fe10efb57a99e1.png)

Procedemos a realizar un fuzzing de directorios
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x html,php,txt,py,sh,log,js,zip -t 200 -U httpadmin -P fhttpadmin
```
![Pasted image 20260130143111.png](../../_assets/da207a1ea7a993420ae4279404a0cac27fe5f84dbcdf539960b2d3b321a21d67.png)

Nos dirigimos a `/login.php`
![Pasted image 20260130141703.png](../../_assets/bc97796156cad2e927b9416097ab6f60e23deb87d8abb51946451fd02f36786c.png)
Y vemos que hay un formulario de logín

Interceptamos la pagina con `burpsuite`
![Pasted image 20260130142821.png](../../_assets/5516d7510323f441ff061201cb45ae441adf5c06fbf3b25cb869de1d70023d9c.png)
Y vemos que realiza una petición `POST`

Entonces lo que se decide hacer es una fuerza bruta con el usuario `admin`
```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 172.17.0.2 http-post-form "/login.php:username=^USER^&password=^PASS^:H=Authorization: Basic aHR0cGFkbWluOmZodHRwYWRtaW4=:F=Credenciales incorrectas." -t 64
```
![Pasted image 20260130145519.png](../../_assets/72e631bfc4412cbbd9b72e8eb703b21deaa66f97fe5e9ec098a921e093362bf9.png)
Y se logro encontrar la contraseña de admin por medio de la fuerza bruta.


Ingresamos las credenciales 
![Pasted image 20260130145722.png](../../_assets/e7f0ab2efa9afdb79b2123b359602c0e814bfe6eccb27a946c780fe4210119ea.png)
Y lo único que nos sale es un mensaje dando la horabuena y un nombre de usuario

Procedemos a realizar una fuerza bruta con aquel usuario mencionado al servicio SSH
![Pasted image 20260130145825.png](../../_assets/0d06e84b3343b877498e5f3300b0d3adab4cb959d0e2de55aca41439cdfc5d7c.png)
Y logramos encontrar su clave

Accedemos al servicio SSH con las credenciales encontradas
![Pasted image 20260130145921.png](../../_assets/5ddaaf29fb05d97c1f40fb87fe6a19bf5992e1935ea46a2be6cab1e4cf9e6d83.png)

No le logro encontrar nada por lo que se procedió también a realizar una fuerza bruta
Para ello se uso el siguiente repositorio de github
https://github.com/Maalfer/Sudo_BruteForce

Lo pasamos de nuestra maquina atacante a nuestra maquina victima.
```bash
scp ./Linux-Su-Force.sh balutin@172.17.0.2:/tmp/Linux-Su-Force.sh

scp /usr/share/wordlists/rockyou.txt balutin@172.17.0.2:/tmp/rockyou.txt
```

Le damos permisos de ejecución y lo ejecutamos
![Pasted image 20260130152441.png](../../_assets/86942f83dac1acb621b35ce11162b3bf63fa408805ad86badfc9124026de696a.png)

Después de un largo tiempo logro encontrar la contraseña de **root**
![Pasted image 20260130154834.png](../../_assets/4e0fde342bfe870eecd313ceca8f97c1eaec8dba61557419a6f8a09b1d04406c.png)

Y logramos acceder como el usuario **root**
![Pasted image 20260130154843.png](../../_assets/a6019197d69a6a72ace6d246a753d552f4df492da3181c6a39bdf56a68a824e4.png)


