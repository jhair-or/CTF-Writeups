
**IP**:  172.17.0.2
**S.O**: Linux
**Source:** DockeLabs
**Level:** Fácil
**Date:** 25 enero 2026
**Autor**: RSA
**Descripción**: 

SQLi
Fuzzing with ffuf
Information Leakage Logs
Enumeración de procesos y comandos 
[[Escalada de Privilegios#**Escala de Privilegios mediante Cron Jobs]]

**Flags**:  


**Begin**:

---

Comenzamos haciendo un ping:
![Pasted image 20260124145203.png](../../_assets/08020fc0a995e0d1671e8fb816fb41898f7fe2f364d3f1292f727711d1a3c924.png)
Hay conexión 
TTL: 64 -> Linux

Escaneo de Puertos abiertos TCP
![Pasted image 20260124145226.png](../../_assets/9545651368e0de5daed5e86cb377f17965f84e417e82f0ebafea7f10997d2486.png)
Puertos abiertos: 
Puerto 22 (SSH)
Puerto 80 (HTTP)

Escaneo de versión al puertos específico 22,80. 
![Pasted image 20260124145251.png](../../_assets/69eed559480e3761d091478849e63240181702c15d4f05df88c3e5cdd85df8c3.png)


Accedemos a la pagina web
![Pasted image 20260124145326.png](../../_assets/da899ccb70e4475b34fbf324fe957fbfbe98d73995c058a636debf5f498fa67b.png)
Y vemos un formulario de login simple.
Primero probamos con si es vulnerable a un sqli

Tras probar logramos acceder con
`' or '1'='1'; -- -`
![Pasted image 20260125191550.png](../../_assets/919fbe206585805c10213340e6976a8e2048e4cd10e32d7c75e863cbf080faa4.png)
Y vemos un mensaje que dice: `System error logs are exposed to the public folder`.

Y al analizar el código fuente vemos un comentario que dice lo siguiente
![Pasted image 20260125191925.png](../../_assets/53cde6fd34470e9f827dcd3d912382c1e5d909406990ac457c3ead48a14aedef.png)

Entonces procedemos a hacer un fuzzing de directorios para encontrar la ruta a `logs.txt`
```bash
ffuf -w /usr/share/wordlists/dirb/common.txt -u "http://172.17.0.2/index.php?page=FUZZ/logs.txt" -H "Cookie: PHPSESSID=icgcpmdfmmue6en7l92b181chh" -fs 43
```
![Pasted image 20260125195820.png](../../_assets/a5f204d25a446f32c3ffc2bc80d3b3c7981603ceef98b9d94d5467b2b4f5eb63.png)
Encontramos el direcctório `logs`

Nos dirigimos a ella y vemos múltiples credenciales y vemos una credenciales que se inicio correctamente que es el del usuario `albert`
![Pasted image 20260125195852.png](../../_assets/c066804dd10c819d85528d9379f00e43184fe581a51da4161d2839af73250c13.png)

Accedemos al usuario `albert` con las credenciales expuestas
![Pasted image 20260125200930.png](../../_assets/0499b52fd51bd4fbe68f9aa30e2058624ab3f6a3943af5379b87798fe4bbeb0e.png)
![Pasted image 20260125200940.png](../../_assets/a3b3d41ee826c67f4dae6c90c3346c2a4e31c0cd5f0ba7927adb055062852309.png)

Una vez dentro, revisamos los procesos que se están ejecutando y vemos que el usuario `conx` ha creado un socket que esta en escucha en `/home/conx/.cache/.sock`, al ser conectado, ejecuta directamente `/bin/bash`.
![Pasted image 20260125210142.png](../../_assets/ad14cedaf4ce453d969b0db2299a81a1406900d3952c3027c85b7b661e916bd3.png)
```bash
socat - UNIX-CONNECT:/home/conx/.cache/.sock
```

Entonces nos conectaremos a ese socket
![Pasted image 20260125205022.png](../../_assets/0b1f2102793bdc5a631b6ea8169e3d07ae043f9e07a906c152b04aecf554596a.png)
Y nos da un `/bin/bash` como el usuario `conx`.

Revisamos las tareas programadas de cron 
![Pasted image 20260125213022.png](../../_assets/ceea335774b992285cb7ba7f83c774d2b97e5818906aa2dd42d4a1b494addce7.png)
Leemos `backup-cron`

Y vemos que se esta ejecutando una tarea un script `backup.sh` por root todo el tiempo
![Pasted image 20260125212955.png](../../_assets/9e5f47f18357abf0097eb80612505bfdfb4ed1ec36b532b71cb595762a0ba0d7.png)

Nos fijamos si tenemos permisos de edición.
![Pasted image 20260125213152.png](../../_assets/791cd0286dc2d72cd898d8e550384f8ddf906222719017154227c33d827fdfec.png)
Y tenemos permisos de escritura

Entonces hacemos que el script nos de permisos suid a la `/bin/bash`
```bash
echo "chmod +s /bin/bash" > /var/backups/backup.sh
```
Y al ejecutarse nos dara uns Shell con permisos de **root**

Luego invocamos la shell con el modo privilegiado
![Pasted image 20260125213504.png](../../_assets/dae2e67051e17ab10f18c777f7eb22df196532110d6dea71cb4565e48560b5d2.png)
Y logramos escalar a **root.**
