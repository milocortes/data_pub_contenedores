# Apptainer

[Apptainer](https://apptainer.org/) es una tecnología de contenedores que simplifica la creación y ejecución de contenedores. Apptainer representa una alternativa a Docker en cómputo cientíico.

La instalación de Apptainer puede encontrarse en la siguiente URL:

* [https://apptainer.org/docs/admin/main/installation.html](https://apptainer.org/docs/admin/main/installation.html)

## Descarga e interacción de imágenes pre-construidas

Podemos usar los comandos ```pull``` y ```build``` para descargar imágenes de algún recurso externo como un registro OCI ([Open Container Initiative](https://opencontainers.org/)).

Podemos descargar imágenes preconstruidas de repositorios como:

* https://hub.docker.com
* https://quay.io

 > Ejemplo de descarga de Docker Hub : ```apptainer pull docker://alpine```

 > Ejemplo de descarga de Quay: ```apptainer pull docker://quay.io/jitesoft/alpine```


También podemos usar el comando ```build``` para descargar imágenes de un recurso externo. Cuando se usa ```build``` se debe especificar el nombre del contenedor:

```bash
apptainer build alpine.sif docker://alpine
```

## Interacción con las imágenes

Hagamos pull a la imagen ```lolcow_latest.sif``` de ghcr.io:

```bash
apptainer pull docker://ghcr.io/apptainer/lolcow
```

### Shell

El comando ```shell``` permite generar un nuevo shell dentro del contenedor e interactuar con él como si fuera una máquina virtual.

```bash
$ apptainer shell lolcow_latest.sif
```

Dentro del contenedor de Apptainer, se mantiene el mismo usuario que el del host:

```bash
Apptainer> whoami
milo
Apptainer> id
uid=1000(milo) gid=1000(milo) groups=1000(milo),65534(nogroup)
```

 > Se agrega ```Apptainer>``` al inicio del shell hace explícito que estamos en un nuevo shell dentro del contenedor.


Podemos agregar una bandera a la instrucción ```apptainer shell``` con un nuevo PID.


### Comando ```exec```

El comando ```exec``` nos permite ejecutar un comando dentro de un contenedor especificando el archivo ```.sif```. Por ejemplo, para ejecutar el programa ```cowsay``` dentro del contenedor ```lolcow_latest.sif```:

```bash
$ apptainer exec lolcow_latest.sif cowsay "Bienvenidos al Data Pub de Mayo!!"
```

### Comando ```run```

Los contenedores de Apptainer pueden contener runscripts. Estos son programas definidos por el usuario donde se especifica las acciones que el contenedor debe llevar acabo cuando se ejecuta. 

El runscript se puede activar con el comando ```run```  o invocando al contenedor como si fuera un ejecutable.

```bash
$ apptainer run lolcow_latest.sif
______________________________
< Mon Aug 16 13:01:55 CDT 2021 >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

$ ./lolcow_latest.sif
______________________________
< Mon Aug 16 13:12:50 CDT 2021 >
 ------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

### Trabajando con archivos
Archivos en el host son alcanzables desde dentro del contenedor:

```bash
$ echo "Hola dentro del Contenedor, Parroquian@s!" > $HOME/hostfile.txt

$ apptainer exec lolcow_latest.sif cat $HOME/hostfile.txt

Hola dentro del Contenedor, Parroquian@s!
```

El ejemplo funciona porque el archivo ```hostfile.txt``` existe en el directorio home del usuario (```$HOME```). Por default, Apptainer monta el directorio ```$HOME```, el directorio de trabajo actual, así como ubicaciones adicionales del sistema del host en el contenedor. 

Es posible montar directorios adicionales en el contenedor con la opción ```--bind```. En el siguiente ejemplo montamos el directorio ```/data``` del host en el directorio  ```/mnt``` dentro del contenedor.

```bash
$ echo "Drink milk (and never eat hamburgers)." > /data/cow_advice.txt

$ apptainer exec --bind /data:/mnt lolcow_latest.sif cat /mnt/cow_advice.txt
Drink milk (and never eat hamburgers).
```

Pipes y redireccionamientos también funcionan con los comandos de Apptainer, exactamente igual que con los comandos de Linux:

```bash
$ cat /data/cow_advice.txt | apptainer exec lolcow_latest.sif cowsay
 ________________________________________
< Drink milk (and never eat hamburgers). >
 ----------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

### Configuración de imagen mediante un sandbox

Un ```sandbox``` es un contenedor dentro de un directorio. 
> Un sandbox es un entorno aislado de pruebas.

Los pasos para configurar una imagen mediante un ```sandbox``` son los siguientes:

* 1) Construir la imagen como ```sandbox``` :
```bash
apptainer build --sandbox <DIR> | <imagen>
```
* 2) Solicitar un shell en el directorio del ```sandbox```:
```bash
apptainer shell --writable <DIR>
```
* 3) Realizar la configuración requerida.
* 4) Convertir el sandbox a formato SIF:
```bash
apptainer build imagen.sif <DIR>
```

La siguiente instrucción construye un sandbox a partir de una imagen de Ubuntu de Docker

```bash
$ apptainer build --sandbox ubuntu/ docker://ubuntu
```

El comando crea un directorio llamado ```ubuntu/``` con un Sistema Operativo completo de Ubuntu y algunos metadatos de Apptainer en el directorio de trabajo actual.

Es posible utilizar comandos como ```shell```, ```exec``` y ```run``` con este directorio sandbox tan cual como lo haríamos con una imagen .sif de Apptainer. Si agregamos la opción ```--writable``` o ```-w``` al utilizar el contenedor, podemos escribir archivos dentro del directorio sandbox (siempre y cuando se cuente con los permisos para hacerlo).

> Si agregamos las opciones ```-pf```, significa que ejecutamos el contenedor en un nuevo PID (```--pid```) así como tambien con la apariencia de ejecutarse como root(```--fakeroot```) 

```bash
$ apptainer exec --writable ubuntu touch /foo

$ apptainer exec ubuntu/ ls /foo
/foo
```

### Convertir imágenes de un formato a otro

El comando ```build``` nos permite construir un nuevo contenedor a partir de un contenedor existente. Por ejemplo, si tienes configurado un directorio sandbox y quieres convertirlo a .sif ( Singularity Image Format), puedes ejecutar la instrucción:

```bash
$ apptainer build new.sif sandbox
```

### Archivos de definición de Apptainer

Para construir un contenedor reproducible, verificable y con calidad de producción, se recomienda crear un archivo SIF utilizando un archivo de definición de Apptainer. Además, facilita la adición de archivos, variables de entorno e instalación de software personalizado. Es posible comenzar con imágenes base de Docker Hub y usar imágenes directamente de repositorios oficiales como Ubuntu, Debian, CentOS, Arch y BusyBox.

Un archivo de definición tiene un encabezado y un cuerpo. El encabezado determina el contenedor base de inicio. El cuerpo está dividido en secciones que realizan tareas como instalación de software, configuración del ambiente, copiado de archivos del host al contenedor, etc. 

El siguiente es un ejemplo de un archivo de definición:

```bash
BootStrap: docker
From: ubuntu:22.04

%post
   apt-get -y update
   apt-get -y install cowsay lolcat

%environment
   export LC_ALL=C
   export PATH=/usr/games:$PATH

%runscript
   date | cowsay | lolcat

%labels
   Author Alice
```

Asumiendo que el archivo de configuración toma el nombre de ```lolcow.def```, podemos construir el contenedor con la instrucción ```build```:

```bash
$ apptainer build lolcow.sif lolcow.def
```

En este caso, el encabezado indica a Apptainer que utilice como imagen base un Ubuntu 22.04 de Docker Hub.

Las secciones definidas en el archivo ejecutan las siguientes tareas:

* ```%post```: esta sección es ejecutada dentro del contenedor durante la construcción del mismo, después que el Sistema Operativo base ha sido instalado. Esta sección es el lugar para realizar instalaciones de nuevas bibliotecas y aplicaciones.
* ```%environment``` : define variables de entorno que estarán disponibles para el contenedor en tiempo de ejecución.
* ```%runscript``` : La sección define las acciones que debe realizar el contenedor cuando es ejecutado con la instrucción ```run```. (Estos comandos no se ejecutarán en el momento de la construcción del contenedor).
* ```%labels```: La sección permite agregar metadatos personalizados al contenedor.
