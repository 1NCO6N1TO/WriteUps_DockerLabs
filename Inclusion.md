- Tags: #LFI #fuerzaBruta 
______
comenzamos la maquina con el escaneo típicos de nmap en el cual conoceremos los puertos abiertos.

```
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.10.10.2 -oG allport
```

para posteriormente aplicar el escaneo de versiones y servicios del mismo nmap por lo que continuemos.

```
nmap -p 80,22 10.10.10.2 -sCV -oN target 
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
![](attachment/89b8cbfc7ae230f976f3ca22c0886571.png)
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
y es así que logramos completar la maquina 