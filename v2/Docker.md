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
