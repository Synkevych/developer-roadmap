## Docker

### Containers

Контейнеризація - наступний крок в еволюції розробці коду.

Контейнери структуровані шарами:
- container layer, самий верхній шар недовговічний і призначений для читання та запису видаляється після видалення 
- base image layers, містять систему та основні залежності тільки для читання

Коли ви створюєте контейнер то замість копіювання всього зображення тільки створюється шар з змінами які ви внесли.

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
