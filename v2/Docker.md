## Docker

### Containers

Контейнеризація - наступний крок в еволюції розробці коду.

Контейнери структуровані шарами:
- container layer, самий верхній шар недовговічний і призначений для читання та запису видаляється після видалення 
- base image layers, містять систему та основні залежності тільки для читання

Коли ви створюєте контейнер то замість копіювання всього зображення тільки створюється шар з змінами які ви внесли - це зменшує розмір архіву та спрощує структуру і можливість оновлення чи змін. 

Переваги:
- Забезпечують сумістність на різних машинах завдяки гіпервізору,
- Не потребують оновлення залежностей та ядра
- Легке масштабування 
- Ізолювання простору користувача для запуску програми
- Їх швидше запустити та видалити 

### Kubernetes

A popular container management and orchestration solution that automates deployment, scaling, load balancing, monitoring and other.
Support stateless applications such as an Nginx or Apache web server and stateful application, butched jobs and daemon tasks.

#### Pods are the core of Kubernetes

Pods represent and hold a collection of one or more containers. Generally, if you have multiple containers with a hard dependency on each other, you package the containers inside a single pod.

![Pods](https://cdn.qwiklabs.com/tzvM5wFnfARnONAXX96nz8OgqOa1ihx6kCk%2BelMakfw%3D)

Pods also have Volumes. Volumes are data disks that live as long as the pods live, and can be used by the containers in that pod. Pods provide a shared namespace for their contents which means that the two containers inside of our example pod can communicate with each other, and they also share the attached volumes.

Pods also share a network namespace. This means that there is one IP Address per pod. By default, pods are allocated a private IP address and cannot be reached outside of the cluster.

#### Services provide stable endpoints for Pods

![Services](https://cdn.qwiklabs.com/Jg0T%2F326ASwqeD1vAUPBWH5w1D%2F0oZn6z5mQ5MubwL8%3D)

Services use labels to determine what Pods they operate on. If Pods have the correct labels, they are automatically picked up and exposed by our services.

The level of access a service provides to a set of pods depends on the Service's type. Currently there are three types:

ClusterIP (internal) -- the default type means that this Service is only visible inside of the cluster,
NodePort gives each node in the cluster an externally accessible IP and
LoadBalancer adds a load balancer from the cloud provider which forwards traffic from the service to Nodes within it.

### GCE 

Google Kubernetes Engine is Kubernetes as a managed service.

**Nodes** the virtual machines that host your containers inside of a GKE cluster
**Pods**

### Commands

```sh
docker ps -a
docker build -t node-app:0.1 .
docker run -p 4000:80 --name my-app node-app:0.1
docker run -p 8080:80 --name my-app-2 -d node-app:0.2 # run as a background process 
docker stop my-app && docker rm my-app
docker logs -f [container_id]
docker exec -it [container_id] bash
docker tag node-app:0.2 gcr.io/[project-id]/node-app:0.2 # tag gcr project
docker push gcr.io/[project-id]/node-app:0.2 # push container to Container Registry
```
