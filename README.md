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

![image](https://user-images.githubusercontent.com/4985062/170286299-577829a1-aab8-4306-a773-c2dc318c147c.png)

3. Antes de automatizar el proceso, vamos a asegurarnos de subir una imagen al repositorio desde nuestro ambiente local. Para ello debemos autorizarnos en ECR utilizando el siguiente comando:

```bash
 $ aws ecr get-login-password --region eu-west-1 --profile [profile] | docker login --username AWS --password-stdin [accountid].dkr.ecr.[region].amazonaws.com
```

Donde:
- `profile` corresponde al profile de AWS configurado a la cuenta actual
- `account-id` es el id de la cuenta de AWS donde queremos subir nuestra solución
- `region` es la región de AWS donde se encuentra nuestro registro del paso anterior

Si el comando es ejecutado con éxito veremos un mensaje de "Login Succeeded" en consola.

3. Es momento de buildear nuestra imagen docker (awesome-api es el nombre de nuestro servicio):

```bash
$ docker build -t awesome-api .
```

4. Con lo anterior en su lugar, podemos finalmente subir la imagen a ECR con el siguiente comando:

```bash
$ docker push [account-id].dkr.ecr.eu-west-1.amazonaws.com/awesome-api:lates
```
5. Si los comandos sucedieron con éxito podremos verificar que la nueva versión es listada en ECR en la Web Console

6. Ahora es momento de comenzar a crear la infraestructura donde esta imagen de Docker será ejecutada. Dirigirse a Menú Services -> Elastic Container Service y crear un nuevo Clúster. Seleccionar la opción de Fargate (Network Only)

![image](https://user-images.githubusercontent.com/4985062/170290064-ea1214c6-4438-48bd-ad21-5dc7265ae037.png)




