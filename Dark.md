- Tags: #pivoting #chisel #socks5 #curl
___
comenzamos la maquina aplicando un reconocimiento de puerto con nmap.

encontramos que los puertos 80 y 22 están abiertos por lo que investigamos la pagina web para ver que tenemos.

____
![](attachment/f9abddd98bbd2cee1065f0978c29a404.png)
___
tenemos la siguiente pagina que nos dice que ingresemos una url, haciendo la prueba no nos arroja nada, por lo que aplicaremos fuzzing de directorios para ver que encontramos.
_____
![](attachment/adec15b9e3df7415e7f9ed5a705a69c5.png)
____
encontramos un directorio llamado info, si accedemos al mismo veremos la siguiente información.
___
![](attachment/33a4cf58c940141a76b2205e76d9fe6b.png)
____
tenemos a un supuesto usuario, llamado toni, ademas tenemos una ip de un segmento de red diferentes. 

para este usuario aplicaremos fuerza bruta utilizando hydra para ssh.
_____
![](attachment/30d8100440d7ee4bc33b14059a225944.png)
____
y tenemos una credencial valida para ssh, por lo nos conectaremos a la maquina y buscaremos pivotar a otra maquina.
_____
![](attachment/9c9c8493f1331ed77afff1d416349a9f.png)
_____
si aplicamos un hostname veremos que la maquina victima tiene dos segmentos de red, el segundo segmento nos servirá como puente para el pivoting.

por mas que busquemos escalar privilegios en la maquina, no sera posible. por lo que igualmente el pivoting lo podemos hacer sin llegar a ser root.

para el pivoting utilizamos chisel, junto con el proxychains para configurar el túnel, ademas de usar foxyproxy para poder acceder a la web del segmento de red al que pivotemos.

## Pivoting 
______
empezamos descargando el chisel, una vez descargado utilizaremos el comando scp para subir el archivo por ssh al directorio /tmp.

podemos aplica un **upx** para bajar un poco el tamaño del chisel y así facilitar su transferencia.

configuramos el cliente de chisel del lado de la maquina victima y el servidor del lado de la maquina atacante.
____
![](attachment/6ecd5416fdb13b751507edca3100272f.png)
____
debemos configurar el archivo de proxychains y agregar el socks5, eso lo hacemos en el directorio **/etc/proxychains.conf**

ademas debemos configurar el foxyproxy para que nos muestre la pagina web del segmento de red en el que estamos.
_____
![](attachment/d47c33a091ea46fb4fe70577abbb5b66.png)
____
ahora si podemos acceder a la pagina web y ver que tal, accediendo a la ip que nos encontramos al principio.
____
![](attachment/45fc349c62a02160e8054aa16d5ac226.png)
____
si aplicamos un curl a esta dirección URL veremos lo siguiente.
____
![](attachment/8eb03915c3842749b96370c0c544b01b.png)
_____
tenemos un parámetro cmd lo cual no es muy habitual, verifiquemos comandos en la pagina para ver que tal.
____
![](attachment/47a008d5fffadfc38e03dd29222559b7.png)
____
aplicamos el comando id y vemos la salida del comando, es hora de aplicar una reverse shell pero no a nuestra maquina, si no al segmento de red 20.20.20.2.
____
![](attachment/d032c9cea6457b75a8796f1f263466e4.png)
___
ganamos acceso a la maquina y aplicamos el tratamiento de la tty ahora busquemos la forma de escalar privilegios.
____
![](attachment/d95d6e62e2d199281767838359ce72b6.png)
____
vemos que curl tiene permisos SUID por lo que alteraremos el archivo passwd el cual esta alojado en el directorio /etc. 

para esto haremos una copia del archivo al directorio tmp y lo modificaremos.

después aplicaremos el siguiente comando:

```shell
curl file:///tmp -o /etc/passwd
```

posteriormente aplicaremos un cat al archivo en el directorio /etc y verificaremos que se hayan guardado las modificaciones, para posteriormente aplicar un `su root`. 
___
![](attachment/d381ce06799ada0343acffe63ee80e76.png)
___
vemos que si hemos retirado la x por lo que ya tendríamos usuario root 