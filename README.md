# TryHackMe - Lazy Admin WriteUp

Walkthrough de la máquina Lazy Admin de TryHackMe. A través de este documento se expondrán los pasos seguidos para vulnerar y completar la máquina Lazy Admin.

## Índice

- [Recon](#recon)
- [Gain Access](#gain-access)
- [Escalate](#escalate)
- [Looting](#looting)

## Recon

Comenzamos escaneando la IP de la máquina con nmap, donde encontramos un servidor web y el puerto SSH abierto.

![NMAP](img/1-nmap.png)

Accedemos al navegador, e introducimos la IP de la máquina seguida del puerto 80, y nos topamos con la página por defecto de Apache.

![APACHE-DEFAULT-WEBPAGE](img/2-Apache-Web-Browser.png)

Como ya sabemos que tiene un servidor web hosteado en la VM, usaremos **gobuster** para descubrir posibles directorios ocultos.

![GOBUSTER-1](img/3-gobuster.png)

El directorio /content está disponible, de modo que lo revisamos personalmente.

![CONTENT-DIRECTORY](img/4-content-directory.png)

Vemos que tiene instalado un servicio web conocido como *Sweetrice*. A partir de aquí, volveremos a usar gobuster, pero partiendo del directorio */content*.

![GOBUSTER-2](img/5-gobuster-2.png)

Si revisamos los directorios */as* y */inc*, nos encontraremos con un formulario de inicio de sesión y un conjuntos de ficheros.

![AS-LOGIN](img/6-content-as-directory.png)

![INC-FILES](img/6-content-inc-directory.png)

Si analizamos con el comando **strings** el fichero **mysql_backup**, notaremos la existencia de un usuario **manager** seguido de un **hash**. Si desciframos este hash puede que encontremos la contraseña del usuario *manager*.

![MYSQL-BACKUP](img/8-strings-mysql.png)

![HASH-PASSWORD](img/9-cracked-hash.png)

Ya tenemos el usuario **manager** y la contraseña **Password123**, las cuales emplearemos en el formulario de inicio de sesión alojado en */as*. Una vez dentro, activaremos la página web oprimiendo el botón *Running*. 

![AS-LOGIN-SUCCESFUL](IMG/10-SweetRice-WebPage.png)

![SWEETRICE-HOME](img/11-SweetRice-Home.png)

## Gain Access

Como sabemos que la versión de SweetRice es la 1.5.1, buscamos un exploit para dicha versión. Encontramos un exploit de tipo Cross-Site Request Forgery que emplea la ejecución de código PHP, por lo tanto lo descargamos.

![1.5.1-EXPLOIT](img/12-searchsploit.png)

![EXPLOIT-DOWNLOAD](img/13-cat-40700-html.png)

Según lo mostrado en el fichero del exploit, debemos ir a la pestaña Ads de la página web, donde subiremos el código PHP malicioso.

![PHP-1](img/14-PHP1.png)

La sección en PHP que se nos proporciona será cambiada por la que vemos en [este repositorio de Github](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php). Donde pone IP, obviamente pondremos la dirección IP de nuestro host.

![PHP-2](img/14-PHP2.png)

![PHP-3](img/14-PHP3.png)

Oprimiremos el botón *DONE* y subiremos el fichero, el cual podremos ver en el directorio */content/inc/ads*. Luego, escucharemos con **Netcat** en el puerto **1234**, que es el que pusimos en el fichero PHP. Obviamente nos aseguraremos de estar escuchando antes de ejecutar el archivo.

![CONTENT-INC-ADS](img/15-content-inc-ads.png)

![SHELL](img/17-shell.png)

Mejoraremos la shell obtenida con el comando de Python siguiente: <br>
`python -c 'import pty; pty.spawn("/bin/bash")'`.

![SHELL-2](img/17-shell-2.png)

Si navegamos por el directorio */home/itguy* encontraremos la primera flag en el archivo **user.txt**. Si ejecutamos el comando `sudo -l` se nos informará que podemos modificar el archivo **backup.pl** sin necesidad de una contraseña. 

![SUDO-L](img/18-sudo-l.png)

## Escalate

Si miramos el contenido del script en Perl mencionado arriba notaremos que se referencia otro script en bash denominado como **copy.sh**. Lo alteraremos para poder obtener una shell con privilegios de root, ya que este script pertenece a dicho usuario. Una vez alterado, lo ejecutaremos desde el primer script en Perl, que SÍ podemos ejecutar directamente.

![CAT-BACKUP-PL](img/19-cat-backup-pl.png)

![SUDO-PRIVILEGES](img/20-sudo.png)

## Looting

Una vez obtenidos privilegios de administrador, buscaremos la última bandera. Para ello, iremos al directorio *root*, donde se encuentra el archivo **root.txt**.

![ROOT.TXT](img/21-root.png )