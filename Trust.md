- tags: #ssh #pivoting #vim 
___
comenzamos aplicando el escaneo básico de nmap en este caso con el proxychains porque esta maquina se puede resolver normal o en un laboratorio de pivoting es lo mismo que el laboratorio normal, lo único que cambia son los comandos y sus sintaxis del lado del atacante.
___
![](attachment/e181f9080e94d7a87f37f47ef4368855.png)
_____
tenemos dos puertos abiertos, procedemos a aplicar los scripts basicos de reconocimiento.
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

