<div align="center">
         <img width="738" height="246" alt="imagen" src="https://github.com/user-attachments/assets/588bd697-af99-459a-99a3-82ce6ec6d583" />
</div>

# Demo de Podman

## Requisitos previos
Podman debe estar instalado en tu máquina. Para ello, usaremos Podman Desktop. Andá a [podman-desktop.io](https://podman-desktop.io/) para descargar e instalar la aplicación.

Dependiendo de tu sistema operativo, habrá pasos diferentes para instalar y ejecutar el comando "podman".

También recomendamos usar VSCode como IDE. Podés descargarlo [aquí](https://code.visualstudio.com/).

Por último, necesitarás tener una cuenta de Red Hat. Podés crear una [aquí](https://access.redhat.com/login).

## Configuración de la tarea
Dentro de VSCode, abrí una terminal y ejecutá el siguiente comando:
```bash
git clone https://github.com/ryangniadek/podman-demo.git
```

Luego abrí la carpeta en VSCode.

## Introducción
El objetivo de este laboratorio es aprender:
- Qué es un contenedor y en qué se diferencia de desplegar de forma nativa o en una máquina virtual
- Cómo ejecutar una imagen de contenedor desde un registro público
- Cómo construir una imagen de contenedor
- Cómo subir una imagen de contenedor a un registro público

Los contenedores son una forma, independiente del entorno, de empaquetar código, configuración y dependencias, y ejecutarlos en cualquier lugar donde haya un motor de contenedores (como Podman o Docker) instalado.

Tanto los contenedores como las máquinas virtuales son formas de proporcionar aislamiento de recursos y están muy extendidos en los despliegues de aplicaciones modernas. Las máquinas virtuales abstraen el hardware y emulan el sistema operativo completo. Por otro lado, los contenedores usan el sistema operativo subyacente del host para ejecutarse, por lo que son mucho más livianos, ya que solo necesitan incluir el código y las dependencias de una aplicación específica.

## Cómo ejecutar una imagen de contenedor
En esta sección, ejecutaremos una imagen de contenedor desde un registro público. Usaremos el registro [quay.io](https://quay.io/), que es un registro público alojado por Red Hat.

Dentro de tu terminal, escribí el siguiente comando:
```bash
podman run quay.io/podman/hello
```
Tu salida debería verse parecida a esto:
```bash
Trying to pull quay.io/podman/hello:latest...
Getting image source signatures
Copying blob sha256:6f7d332c6972d7de13acdd07eafee248b5435ff1b69d22d1cbbe9c64198d4777
Copying config sha256:1b10fa0fd8d184d9de22a553688af8f9f8adbabb11f5dfc15f1a0fdd21873db2
Writing manifest to image destination
!... Hello Podman World ...!

         .--"--.
       / -     - \
      / (O)   (O) \
   ~~~| -=(,Y,)=- |
    .---. /`  \   |~~
 ~/  o  o \~~~~.----. ~~
  | =(X)= |~  / (O (O) \
   ~~~~~~~  ~| =(Y_)=-  |
  ~~~~    ~~~|   U      |~~

Project:   https://github.com/containers/podman
Website:   https://podman.io
Documents: https://docs.podman.io
Twitter:   @Podman_io
```

Es posible que notes que, antes de la salida del contenedor de "hola mundo", hay algunas líneas que muestran cómo se descarga la imagen del contenedor desde Quay.

## Cómo construir una imagen de contenedor
En esta sección, construiremos una imagen de contenedor usando un Containerfile. Un Containerfile (a veces llamado Dockerfile). Cada línea del archivo es un comando que se ejecutará, en orden, cuando se construya la imagen del contenedor.

> El orden de los comandos es importante, ya que los contenedores usan un sistema de archivos en capas, de modo que cuando se hacen cambios, solo se actualiza la capa que necesita cambiar y las siguientes. Por lo tanto, las capas que rara vez cambian, como las dependencias, deberían ir al principio del Containerfile, y las capas que cambian con frecuencia, como el código de la aplicación, deberían ir al final.

El Containerfile que te proporcionamos se ve así:
```Dockerfile
# Set the base image to the UBI 9 Python 3.11 image provided by Red Hat
FROM registry.access.redhat.com/ubi9/python-311:1-41
# Set working directory inside the container to /app
WORKDIR /app
# Copy the requirements.txt file from the local host to the container's /app directory
COPY ./application/requirements.txt /app
# Install the Python dependencies from the requirements.txt file
RUN pip install -r requirements.txt
# Copy the application files from the local host to the container's /app directory
COPY ./application /app
# Expose port 5000
EXPOSE 5000
# Set the default command for the container to flask run
CMD flask run --host=0.0.0.0
```

(Los comentarios del archivo, en orden: establece la imagen base como la imagen UBI 9 con Python 3.11 provista por Red Hat; establece el directorio de trabajo dentro del contenedor en /app; copia el archivo requirements.txt del host local al directorio /app del contenedor; instala las dependencias de Python desde requirements.txt; copia los archivos de la aplicación del host local al directorio /app del contenedor; expone el puerto 5000; establece flask run como comando predeterminado del contenedor.)

Este Containerfile usa una imagen base provista por Red Hat, que es un sistema operativo mínimo que incluye Python 3.11. Luego copia los requisitos de una aplicación Flask de ejemplo que te proporcionamos, los instala dentro del contenedor, copia el resto del código de la aplicación, especifica que el contenedor debe escuchar en el puerto 5000 y, por último, establece como comando predeterminado la ejecución de la aplicación Flask.

Ahora construyamos la imagen del contenedor y llamémosla `my_app`. En tu terminal, escribí el siguiente comando:
```bash
podman build -t my_app .
```

Deberías ver una salida que indica cada capa de la imagen del contenedor a medida que se construye.

Para ver la lista de todas las imágenes en tu sistema, ejecutá este comando:
```bash
podman images
```

## Ejecutá tu imagen de contenedor
Ahora que construiste una imagen de contenedor `my_app`, podés ejecutarla con el siguiente comando:
```bash
podman run --rm -d -p 5000:5000 --name api my_app
```
Expliquemos qué hacen las diferentes opciones del comando `podman run`:
- `--rm` elimina automáticamente el contenedor y su sistema de archivos después de detenerlo
- `-d` modo desacoplado (detached), ejecuta el contenedor como un proceso en segundo plano
- `-p` expone un puerto del contenedor en la máquina host, dado el argumento `puertoHost:puertoContenedor`. Entonces, en el comando anterior, estamos exponiendo el puerto `5000` dentro del contenedor en el puerto `5000` de la máquina host
- `--name` asigna un nombre a tu contenedor en ejecución, en nuestro caso, `api`. Si no se asigna un nombre, se genera una cadena aleatoria
- `my_app` es el nombre de la imagen de contenedor que se va a ejecutar.

Ahora que el contenedor está en ejecución, podés consultar la aplicación Flask que corre dentro del contenedor ejecutando los siguientes comandos:
```bash
curl localhost:5000
curl localhost:5000/your-name-here
```

Cuando hayas terminado de probar, detené el contenedor:
```bash
podman stop api
```

## Subir tu imagen de contenedor a un registro
Ahora que construiste una imagen de contenedor, podés subirla a un registro. Un registro es un lugar donde se almacenan imágenes de contenedores. Podés pensarlo como un GitHub para imágenes de contenedores.

En este laboratorio, usaremos [quay.io](https://quay.io/), que es un registro público alojado por Red Hat.

Para subir tu imagen de contenedor a quay.io, necesitarás iniciar sesión con tu cuenta de Red Hat. Podés hacerlo [aquí](https://quay.io/).

Una vez que hayas iniciado sesión, podés hacer clic en "Create New Repository" y darle un nombre. Para este laboratorio, usaremos `my_app`. Hacé que el repositorio sea público, seleccioná la opción de repositorio vacío y hacé clic en "Create Public Repository".

Ahora que creaste un repositorio, podés subir tu imagen de contenedor a él. Primero, tendrás que iniciar sesión en quay en tu máquina local. Para ello, ejecutá el siguiente comando e ingresá tus credenciales de Red Hat cuando se te pidan:
```bash
podman login quay.io
```

Ahora que iniciaste sesión, podés subir tu imagen de contenedor a quay.io. Para ello, tendrás que etiquetar tu imagen de contenedor con el nombre del repositorio que creaste, nombrándola igual que tu repositorio remoto. Ejecutá el siguiente comando para hacerlo:
```bash
podman tag my_app quay.io/<your-username>/my_app
```

Luego podés subir tu imagen de contenedor a quay.io ejecutando el siguiente comando:
```bash
podman push quay.io/<your-username>/my_app
```

Tu imagen de contenedor ya está publicada en el repositorio remoto si obtuviste este mensaje:
```bash
...
Writing manifest to image destination
```




