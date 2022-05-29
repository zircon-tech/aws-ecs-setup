# Deployment a AWS ECS + ECR + Gitlab CI/CD

Este documento describe brevemente los pasos necesarios para ejecutar tu aplicación (ya sea web, api o servicio) compilada en una imagen Docken en la nube de AWS. 
Las imágenes de Docker son almacenadas en Elastic Container Registry de AWS y su ejecución es realizada en Elastic Container Service Fargate. Adicionalmente,
se muestra un flujo de CI/CD en Gitlab para automatizar el deployment al hacer push sobre branches estrategicos.

Se detallan a continuación los pasos necesarios para dejar nuestra app lista en la nube de AWS:

### Prerequisitos

1. Será necesario tener `aws cli` debidamente configurado en el ambiente local. También asegurarse de tener las AWS Keys configuradas correctamente en un profile. Más información sobre cómo configurarlo se puede encontrar [aquí](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html).
2. Instalar Docker Desktop localmente e iniciar sesión con nuestras credenciales por consola utilizando `docker login`

### Procedimiento

1. Acceder a la cuenta de AWS de Zircon [acá](https://670171959034.signin.aws.amazon.com/console). Solicitar usuario de AWS en caso de no tenerlo.
2. . Lo primero que debemos hacer es crear un repositorio privado para nuestras imágenes de Docker. Accedé al Menú Services -> Elastic Container Registry y creá un nuevo repositorio privado, solo es necesario asignar un nombre.

<img src="https://user-images.githubusercontent.com/4985062/170286299-577829a1-aab8-4306-a773-c2dc318c147c.png" width="500"/>

3. Antes de automatizar el proceso, vamos a asegurarnos de subir una imagen al repositorio desde nuestro ambiente local. Para ello debemos autorizarnos en ECR utilizando el siguiente comando:

```bash
 $ aws ecr get-login-password --region eu-west-1 --profile [profile] | docker login --username AWS --password-stdin [accountid].dkr.ecr.[region].amazonaws.com
```

Donde:
- `profile` corresponde al profile de AWS configurado a la cuenta actual
- `account-id` es el id de la cuenta de AWS donde queremos subir nuestra solución
- `region` es la región de AWS donde se encuentra nuestro registro del paso anterior

Si el comando es ejecutado con éxito veremos un mensaje de "Login Succeeded" en consola.

<hr />

4. Es momento de buildear nuestra imagen docker (awesome-api es el nombre de nuestro servicio):

```bash
$ docker build -t awesome-api .
```

5. Con lo anterior en su lugar, podemos finalmente subir la imagen a ECR con el siguiente comando:

```bash
$ docker push [account-id].dkr.ecr.eu-west-1.amazonaws.com/awesome-api:latest
```
6. Si los comandos sucedieron con éxito podremos verificar que la nueva versión es listada en ECR en la Web Console

7. Ahora es momento de comenzar a crear la infraestructura donde esta imagen de Docker será ejecutada. Dirigirse a Menú Services -> Elastic Container Service y crear un nuevo Clúster. Seleccionar la opción de Fargate (Network Only)

<img src="https://user-images.githubusercontent.com/4985062/170290064-ea1214c6-4438-48bd-ad21-5dc7265ae037.png" width="500" />

Asignar un nombre y seleccionar la opción de VPN con sus valores por defecto. Crearemos el resto de los recursos dentro de esta misma VPN.
<br />
<br />
<img src="https://user-images.githubusercontent.com/4985062/170702355-3032cbd4-5ed3-4501-9491-d0113a0ffa49.png" width="500" />
<br />

8. Como parte de este clúster, el siguiente paso es crear una nueva Task Definition. Las definiciones de tareas especifican la información del contenedor para su aplicación, como cuántos contenedores forman parte de su tarea, qué recursos usarán, cómo se vinculan entre sí y qué puertos de host usarán.
<br /><br />

En el menú de Tasks Definitions crear una nueva tarea y seleccionar la opción Fargate que facilitará la provisión de instancias para nuestra aplicación 
<br /><br />
<img src="https://user-images.githubusercontent.com/4985062/170702774-7bf100b3-efec-4240-b8d9-9ef9da2ad2ea.png" width="500"/>
<br /><br />

Asignar un nombre y un sistema operativo
![image](https://user-images.githubusercontent.com/4985062/170702986-13c6d6ea-6122-4348-a544-3688bde6a7be.png)

<br /><br />

Luego proveer los requisitos de hardware para nuestra aplicación, el costo de Fargate estará basado en esto. Para un setup inicial elegir los valores mīnimos:
![image](https://user-images.githubusercontent.com/4985062/170703309-55d95346-dad6-423c-8688-59eb9f59c526.png)

<br /><br />

El siguiente paso es agregar nuestro Container Docker que creamos al comienzo. Para ello asignar un nombre y el ARN a nuestra imagen dentro de ECR.
![image](https://user-images.githubusercontent.com/4985062/170703796-0665333c-155c-4f07-a84f-bbe2bba9b98a.png)

<br /><br />

Dentro de la misma ventana, agregar las variables de entorno que son necesarias para ejecutar la aplicación

<img src="https://user-images.githubusercontent.com/4985062/170703686-031ff2a5-9168-4db4-9327-6883c0806a38.png" width="800" />

<br /><br />

Dejar el resto de los valores por defecto y click en Create.

<hr />

8. Para que nuestro servicio sea accesible y escalable, es recomendable configurar un Load Balancer que sea el punto de entrada a nuestra aplicacion. El LB redirige el trafico a los contenedores Fargate a traves de un Target Group que identifica el pool de instancias el cual se actualiza automaticamente con la administracion de dichas imagenes. Para ello, abandonar la seccion de ECS y dirigirse a EC2 para configurar un Application Load Balancer
![image](https://user-images.githubusercontent.com/4985062/170716590-675cd65c-ed9a-46a8-a0df-2982e14e3bb3.png)

Luego, asignar y nombre y seleccionar la VPC configurada anteriormente.
![image](https://user-images.githubusercontent.com/4985062/170716761-49bf3c60-cb38-47fd-93de-3d7a787bcb7f.png)

Posteriormente, seleccionar uno de los security groups de la VPC y dentro de la sección de listeners proceder a crear un Target Group.
![image](https://user-images.githubusercontent.com/4985062/170717018-22554dd8-8045-4c58-80a5-5bbc8379450d.png)


9. Con nuestra Task Definition en su lugar procedemos a crear un Servicio. Un servicio permite especificar cuántas copias de la definición de tarea ejecutar y mantener en un clúster. Vamos dentro del cluster, tab de Services, Create. Seleccionar un nombre y elegir la Task y Cluster creados en los pasos anteriores. Adicionalmente, seleccionar el numero de Tasks redundantes a ejecutar.

![image](https://user-images.githubusercontent.com/4985062/170713695-8f6e3105-cf87-4784-8ee3-d22732e875f0.png)

![image](https://user-images.githubusercontent.com/4985062/170713916-4e8d51fe-bece-4d2c-821d-f913e75dff9c.png)



Dejar el resto de los valores por defecto y finalizar la creación.

<br/>
<br/>
ECS se encargará de provisionar los recursos y descargar la imagen de ECS para su ejecución. 


