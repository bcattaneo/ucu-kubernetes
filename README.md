# ucu-kubernetes

Prueba uso de Kubernes con balanceador de carga.

Veremos todo el flujo para realizar el deploy y escalabilidad de un servicio (API flask).

Curso "INSTALACIÓN CONFIGURACIÓN Y ADM.DE LINUX" / Universidad Católica del Uruguay

## Imágenes de Docker
En el proceso utilizaremos contenedores de Docker, los cuales enviaremos a Kubernetes para su ejecución.

### Construcción
Primero debemos construír imagen del servicio #1
```
cd service1
docker build -t flask_service1:v0 .
```

Podemos corroborar su funcionamiento ejecutándola:
```
docker run -p 5000:5000 flask_service1:v0
```

### Subir a DockerHub
Para crear nuestras propias imágenes y realizar deploy de las mismas, debemos crear una cuenta en DockerHub: https://docs.docker.com/docker-hub/

Listamos las imágenes creadas disponibles:
```
docker images
```

Obteniendo el ID y tags, creamos:
```
docker tag <ID imagen> <ID dockerhub>/<nombre repo>:<tag>
```
ej: ```docker tag bc52e14d03ca bcattaneo/flask-service1:v0```

Subimos al repositorio:
```
docker push <ID dockerhub>/<nombre repo>:<tag>
```
ej: ```docker push bcattaneo/flask-service1:v0```

## Deployment
  
### Conexión inicial
Lo primero que debemos hacer es conectar "kubectl" con nuestro cluster de Kubernetes. En nuestro caso estamos utilizando uno de DigitalOcean.
  
Una forma de utilizar la herramienta, es proveer la configuración del cluster en cada comando, por ejemplo:
```
kubectl --kubeconfig=~/.kube/<nombre cluster>-kubeconfig.yaml get nodes
```
ej: ```kubectl --kubeconfig=/.kube/ucu-kubernetes-kubeconfig.yaml get nodes```

La forma recomendada de manejar configuraciones es definir la variable de entorno `KUBECONFIG` apuntando a la ruta de nuestra configuración.

Otra forma, cuando sólo vamos a manejar un cluster, es renombrar <clustername>-kubeconfig.yaml a config. De esta forma se utiliza dicha configuración del cluster en todos los comandos.

Creamos un nuevo namespace para nuestro proyecto:
```
kubectl create namespace flask
```

Listamos namespaces actuales para corroborar que existe "flask":
```
kubectl get namespace
```

### Deploy de un pod

El pod es la unidad mínima en Kubernetes.
  
Configuramos uno para el servicio utilizando:
```
kubectl apply -f flask-pod.yaml -n flask
```

Listamos los pods actuales, para corroborar que vemos el nuevo:
```
kubectl get pod -n flask
```
_Nota: podría demorar entre 10 y 20 segundos en visualizarse el nuevo pod_

Una forma sencilla de probar el funcionamiento del servicio, es rediregir el puerto del cluster a uno local:
```
kubectl port-forward pods/flask-pod -n flask 5000:5000 --address='0.0.0.0'
```

En nuestro entorno de desarrollo deberíamos lograr acceder al servicio en el puerto 5000.

### Creando un deployment
  
Normalmente se pretende manejar todo el ciclo de vida de varios contenedores, crear réplicas de los mismos o bien mantener varios pods en un mismo contexto. Con Kubernetes podemos crear deployments.
  
Primero eliminamos el pod individual:
```
kubectl delete pod flask-pod -n flask
```

Realizamos el deploy del servicio con dos réplicas:
```
kubectl apply -f flask-deployment.yaml -n flask
```

Visualizamos los deployment con el siguiente comando:
```
kubectl get deploy -n flask
```

Si revisamos los pods, ahora veremos dos:
```
kubectl get pod -n flask
```

Nuevamente probamos el servicio localmente:
```
kubectl port-forward deployment/flask-dep -n flask 5000:5000
```

### Servicio balanceador

Con Kubernetes es muy sencillo crear un servicio balanceador para nuestro servicio:

```
kubectl apply -f flask-service.yaml -n flask
```

Revisamos estado del servicio:
```
kubectl -n flask get svc -w
```
_Nota: puede tardar unos minutos en iniciarse_

## Conclusiones
Con unos pocos pasos podemos realizar un deploy de un servicio real, con balanceador, control de carga y muchas otras utilizadades que ya vienen "out-of-the-box". Adicionalmente, el dashboard de Kubernetes incluye muchas utilizadades visuales para el monitoreo de nuestros servicios.
  
## Referencias
* https://www.digitalocean.com/community/meetup_kits/getting-started-with-containers-and-kubernetes-a-digitalocean-workshop-kit
* https://github.com/do-community/k8s-intro-meetup-kit.git
  
## Licencia
MIT
