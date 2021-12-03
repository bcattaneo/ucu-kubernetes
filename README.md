# ucu-kubernetes

Curso "INSTALACIÓN CONFIGURACIÓN Y ADM.DE LINUX"

## Construcción imágenes
Construyo imagen del servicio #1
cd app
sudo docker build -t flask_service1:v0 .

Pruebo que funciona:
sudo docker run -p 5000:5000 flask_service1:v0

CREAR CUENTA EN DOCKERHUB, para usar nuestra propia imagen
https://docs.docker.com/docker-hub/

se sube nuestra imagen creada con:
sudo docker images
sudo docker tag <ID imagen> <ID dockerhub>/<nombre repo>:<tag>
ej: sudo docker tag bc52e14d03ca bcattaneo/flask-service1:v0
sudo docker push <ID dockerhub>/<nombre repo>:<tag>
ej: sudo docker push bcattaneo/flask-service1:v0

# Deployment
Conectar kubectl con nuestro cluster de digitalocean
una forma kubectl --kubeconfig=~/.kube/<clustername>-kubeconfig.yaml get nodes
ej.: kubectl --kubeconfig=/.kube/ucu-kubernetes-kubeconfig.yaml get nodes
la otra es renombrar <clustername>-kubeconfig.yaml a config

Creamos namespace
kubectl create namespace flask

Listamos namespaces, debería existir "flask" ahora
kubectl get namespace

Deploy del pod:
kubectl apply -f flask-pod.yaml -n flask

Listamos pods:
kubectl get pod -n flask
Nota: A los 10 segundos aprox.. debería estar ejecutando

Forward del puerto para probar:
kubectl port-forward pods/flask-pod -n flask 5000:5000 --address='0.0.0.0'

Deberíamos poder recibir nuevamente el servicio.

Ahora haremos lo mismo pero con un deploy:
kubectl delete pod flask-pod -n flask

Se realiza el deploy con:
kubectl apply -f flask-deployment.yaml -n flask

Vemos los deploy con:
kubectl get deploy -n flask

Para ver cada pod de forma individual:
kubectl get pod -n flask

Nuevamente, podemos probarla haciendo forward de los puertos:
kubectl port-forward deployment/flask-dep -n flask 5000:5000

Levantamos servicio con balanceador:
kubectl apply -f flask-service.yaml -n flask

Revisamos estado del servicio:
kubectl -n flask get svc -w

TODO: caso donde cambia la imagen


## Referencias
* https://www.digitalocean.com/community/meetup_kits/getting-started-with-containers-and-kubernetes-a-digitalocean-workshop-kit
* https://github.com/do-community/k8s-intro-meetup-kit.git