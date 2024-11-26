## Objetivo
Explorar cómo configurar una acción personalizada de Docker en GitHub Actions.

El objetivo de nuestra acción personalizada de Docker es ejecutar un Saludo por consola en un contenedor Docker  abstraer eso detrás de una acción reutilizable fácil de usar.

## Tareas

1. Crear la carpeta .github/actions/hello-to-you-docker. Esta carpeta es donde alojaremos todos los archivos para nuestra acción personalizada de Docker.
2. Crear un archivo llamado action.yaml en la carpeta .github/actions/hello-to-you-docker.
3. En el archivo action.yaml, añadir las siguientes propiedades:
   - Un 'name' de Hello to You Docker;
   - Un 'description' de "Prints a greeting message in a Docker container";
   - La acción debería recibir un input:
     - El primero, llamado 'name', debería ser requerido, tener un description 'Your name', y por defecto ser 'World'.
   - Añadir una key 'runs' a nivel de workflow. Esta es la clave principal para definir nuestra acción personalizada en Docker. Para una acción personalizada en Docker, la key 'runs' tiene la siguiente forma:
   ```yaml
   runs:
     using: docker
     image: Dockerfile
   ```
   donde:
        - 'using: docker' define que la acción se ejecutará usando Docker.
        - 'image: <Dockerfile o imagen>' define qué archivo se usará para construir la imagen Docker, o qué imagen se usará para la ejecución.
4. Crear un archivo Dockerfile en la carpeta .github/actions/hello-to-you-docker. El Dockerfile debería usar la imagen de node:alpine3.19 como imagen base, copiar un shell script llamado entrypoint.sh al contenedor, y ejecutar el script al iniciar el contenedor. Si no estás familiarizado con Docker y Dockerfile, el código para el Dockerfile:
   ```Dockerfile
   # Set the base image to use for subsequent instructions
   FROM alpine:3.19
   
   # Set the working directory inside the container
   WORKDIR /tmp
   
   # Copy any source file(s) required for the action
   COPY entrypoint.sh .

   # add exec. permissions
   RUN ["chmod", "+x", "/tmp/entrypoint.sh"]
   
   # Configure the container to be run as an executable
   ENTRYPOINT ["/tmp/entrypoint.sh"]
   ```
5. Crear un archivo entrypoint.sh en la carpeta .github/actions/hello-to-you-docker. El script debería imprimir un mensaje de saludo en la consola. Por ejemplo:
   ```bash
   #!/bin/sh
   
   # Use INPUT_<INPUT_NAME> to get the value of an input
    echo "Hello, $INPUT_NAME"
    ```
6. Crear un archivo custom-actions-docker.yaml en la carpeta .github/workflows en la raíz de tu repositorio.
7. Nombrar el flujo de trabajo Custom Actions - Docker.
8. Añadir los siguientes desencadenantes con filtros de eventos y tipos de actividad a tu flujo de trabajo:
    - workflow_dispatch: además, el desencadenador workflow_dispatch debería recibir un input, llamado name, de tipo string y con un valor por defecto de 'World'.
9. Añadir un único trabajo llamado hello-to-you-docker al flujo de trabajo.
10. Debería ejecutarse en ubuntu-latest.
11. Debería contener dos steps:
    - El primer step debería hacer checkout del código.
    - El segundo step, llamado Hello to You, debería usar la acción personalizada de Docker creada en pasos anteriores. Para hacer referencia a una acción personalizada creada en el mismo repositorio que el flujo de trabajo, simplemente puedes proporcionar la ruta del directorio donde se encuentra el archivo action.yaml. En este caso, esto sería ./.github/actions/hello-to-you-docker. Asegúrate de pasar el input name a la acción personalizada.
12. Confirmar los cambios y hacer push del código. Desencadenar el flujo de trabajo desde la interfaz de usuario y tómate unos momentos para inspeccionar la salida de la ejecución del flujo de trabajo.

## Añadir parámetros de salida a la acción personalizada de Docker

1. Modificar el archivo action.yaml para la acción personalizada de Docker:
   - Añadir una key 'outputs' a nivel de acción. Esta es la clave principal para definir las salidas de nuestra acción personalizada.
   - Añadir una salida llamada time, con una descripción de 'The time we greeted you'.
2. Actualizar el script entrypoint.sh para que devuelva la hora actual en la salida de la acción, redirigiendo la variable de entorno GITHUB_OUTPUT al archivo de salida de GitHub.
       ```bash
       #!/bin/sh
         
       # Use INPUT_<INPUT_NAME> to get the value of an input
       echo "Hello, $INPUT_NAME"
         
       # Write outputs to the $GITHUB_OUTPUT file
       echo "time=$(date)" >>"$GITHUB_OUTPUT"
       ```
3. Modificar el flujo de trabajo custom-actions-docker.yaml para capturar la salida de la acción personalizada de Docker.
4. Añadir un nuevo job, llamado Get the time, que debería ejecutar el script de GitHub para capturar la salida de la acción personalizada de Docker, y mostrar la hora en la consola.
