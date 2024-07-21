- Tags: #FTP #javascript #nodejs
____
comenzamos la maquina aplicando el típico escaneo de nmap y nos reporta lo siguiente.
_____
![](attachment/6684151bcd3bf3ca98c91e7e3bab02ed.png)
_____
vemos de una que el puerto 21 (ftp) es vulnerable a utilizar el usuario de invitado anonymous, por lo que nos conectamos al servicio solamente proporcionando el usuario anonymous.
_____
![](attachment/859a86d39f09b7766e73907fa8388cc1.png)
___
vemos que tenemos un comprimido el cual tiene contraseña para poder acceder a su contenido por lo que utilizaremos zip2john para poder extraer su hash y con john poder crakear la contraseña.
_____
![](attachment/c5601b6e58a277b42071b01ac29fd45a.png)
____
tenemos la contraseña, podemos descomprimir el archivo y a ver que tiene en su interior.
___
![](attachment/a1fc382d10ba35cb9c6f0d1426287e35.png)
_____
tiene credenciales que usaremos para conectarnos por ssh.
____
![](attachment/83fa19b975694465c1c65476dab9c2a8.png)
_____
entramos como el usuario mario y vemos que en nuestro directorio existe un script.js y también noto que tiene permisos de escritura.

si aplicamos un sudo -l veremos lo siguiente.
______
![](attachment/4a4da3c4836e14d08eba749e1b8ab3af.png)
____
podemos ejecutar dicho script como super usuario sin proporcionar contraseña, por lo que nos disponemos en cambiar los permisos de la /bin/bash.

el script tendrá el siguiente contenido:

```javascript
const { exec } = require('child_process');

exec('chmod u+s /bin/bash', (error, stdout, stderr) => {
    if (error) {
        console.error(`Error: ${error.message}`);
        return;
    }
    if (stderr) {
        console.error(`Stderr: ${stderr}`);
        return;
    }
    console.log(`Stdout: ${stdout}`);
});
```

_____
ejecutamos el script y aplicamos un bash -p para obtener root.
_____
![](attachment/30d0479a98cbd0f4cd981bcb64b0e1f9.png)
