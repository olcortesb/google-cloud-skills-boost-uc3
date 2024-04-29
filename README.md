# Notas evento BAUC3M & Google and Partners

A continuación notas que compartí con los estudiantes en el transcurso del evento, están divididos por cada laboratorio que trabajamos.

## Task 1. Create a Linux VM instance
- Crear VM instance
  - Console / searchs / VM

> ver el codigo al crear la instancia

```bash
# Codigo para lanzar la isntancia con el comando de gcloud
gcloud compute instances create VM_NAME \
  --image-project=debian-cloud \
  --image-family=debian-10 \
  --metadata=startup-script='#! /bin/bash
  apt update
  apt -y install apache2
  cat <<EOF > /var/www/html/index.html
  <html><body><p>Linux startup script added directly.</p></body></html>
  EOF'
```
> Verificar la version de linux para que los comandos de `apt-get` que se van a ejecutar despues funcionen correctamente

## Task 2. Enable public access to VM instance
- Habilitar HTTP
  - Network / firewall

```bash
# Revisar el estado de los servicios que tenemos levantados
cat /etc/services
grep -w '80/tcp' /etc/services
grep -w '443/tcp' /etc/services
grep -E -w '22/(tcp|udp)' /etc/services

```
## Task 3. Running a basic Apache Web Server
- conectarse por `ssh` a la instancia ( dentro de la misma consola de google cloud)

```bash
#!/bin/bash
apt-get update
apt-get install -y apache2
```

```bash
# Verificar que el apache esta instalado 
dpkg -l | grep apache
```

```bash
# un curl de prueba para ver nuestra ip publica
curl ipv4.icanhazip.com

```

# LAB 2: Scale Out and Update a Containerized Application on a Kubernetes Cluster

### Pasos previos:

- Validar que existe el cluster
- Validar que existe el bucket y el objeto
- abrir consola google shell
- Traer el config para acceder al cluster


```bash
# Traer las credenciales del cluster a la terminal
gcloud container clusters get-credentials echo-cluster --zone=europe-west4-a
```
- Verificar donde esta el config

```bash
cd /home/student_00_eb454a988c14/.kube
ls
more config
```

verificar al version de kubectl
```bash
kubectl version
```
- ver el estado del cluster

```bash
kubectl get nodes -o  wide
kubectl get pods
kubectl get deployment
kubectl get pods -n kube-system
```

- Crear deployment

```bash
# Verificar que deployment tengo
kubectl get deployment
# Editar deployment
kubectl edit deployment/echo-web
```

- (Anexo) Cambiar el editor desde la terminal / abrir el editor web

```
printenv | grep KUBE_EDITOR
export KUBE_EDITOR=nano

```

- Terminar deploy y exponer puerto:

```bash
kubectl create deployment echo-web --image=gcr.io/qwiklabs-resources/echo-app:v1
kubectl expose deployment echo-web --type=LoadBalancer --port 80 --target-port 8000
```

- Verificar que el puerto funcione
  - Ir a: cluster, workload, Exposing services


## Task 1. Build and deploy the updated application with a new tag

- Verificar Versión de `docker`

```bash
docker --version
```

- descargar archivo de google `gs`

```bash
gcloud storage cp gs://qwiklabs-gcp-03-914dea928a54/echo-web-v2.tar.gz .
```

- verificar versión de `tar`

```bash
tar --version
```

- descomprimir archivo

```bash
tar -xf echo-web-v2.tar.gz 
```
- verificar archivos que de descomprimieron 
```bash
cd  echo-web-v2
ls
# Dockerfile  main.go  manifests  README.md
```

- Crear imagen docker verificar imagen y subir imagen

## Task 2. Push the image to the Container Registry
```bash
# El punto "." al final de la primera linea indica el directorio actual
# Suponemos que estamos parados dentro del directorio echo-web-v2 
docker build -t gcr.io/{my-project-id}/echo-app:v2 .
# Verificar que la imagen se creo correctamente
docker images
# Autenticarse en el registry de google
gcloud auth configure-docker
# Subir la imagen
docker push gcr.io/{my-project-id}/quickstart-image
```


## Task 3. Deploy the updated application to the Kubernetes cluster
```bash
kubectl create deployment echo-web-v2 --image=gcr.io/qwiklabs-gcp-00-1ab0d71a448a/echo-app:v2
```

## Task 4. Scale out the application
- Editando el deployment
- Usando `kubectl`
```bash
kubectl scale deployment echo-web --replicas=2
```

## Task 5. Confirm the application is running

- Verificar que esta ejecutándose la version 2

# LAB 3: Deploy a Compute Instance with a Remote Startup Script


## 1. Task 1. Create a storage bucket
   - Crear buckets
  ```bash
  # Los podemos crear también desde la terminal
  gcloud storage buckets create gs://BUCKET_NAME --location=BUCKET_LOCATION
  ```
  - Una ves creado:
  ```bash
  # Copiar un objeto de google Storage con la terminal:
  gcloud storage cp gs://qwiklabs-gcp-03-eb542aca2b0f/resources-install-web.sh .
      
  ```


## 2. Task 2. Create a VM instance with a remote startup script

Dentro del wizard de creación de la instancia ir a:
- Avanzado
  - Management
    - Labels
      - startup-script-url
        - Pegar url del bucket de `gs://`


> Revisar el código que crea la instancia en la barra lateral derecha
>  - https://cloud.google.com/compute/docs/instances/startup-scripts/linux?hl=es-419 
>  - https://cloud.google.com/compute/docs/instances/st>artup-scripts/linux?hl=es-419

## 3. Task 3. Create a firewall rule to allow traffic (80/tcp)
   - Ir a la instancia creada
   - Modificar
   - Agregar http como regla de entrada / solo http 
> Revisar la configuración de network de la instancia

## 4. Task 4. Test that the VM is serving web content
  - Ver la ip publica que crea para el laboratorio

# References:
- https://kubernetes.io/docs/home/
- https://kubernetes.io/docs/concepts/overview/components/
- https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
- https://cloud.google.com/container-registry/docs/advanced-authentication?hl=es-419#gcloud-helper
- https://cloud.google.com/kubernetes-engine/docs/how-to/scaling-apps?hl=es-419#:~:text=To%20use%20kubectl%20scale%20%2C%20you,See%20more%20code%20actions. 
- kubectl: [The Essential Kubectl Commands: Handy Cheat Sheet](https://www.atatus.com/blog/essential-kubectl-commands/#kubectl-apply)
- Startup Script : https://cloud.google.com/compute/docs/instances/startup-scripts/linux?hl=es-419


# ANEXO

Conceptos que usamos o estudiamos en los laboratorios:

- Modelo OSI de referencia o modelo TCP/IP
    - REF: https://www.cloudflare.com/es-es/learning/ddos/glossary/open-systems-interconnection-model-osi/

- Distribuciones Linux:
    - https://upload.wikimedia.org/wikipedia/commons/1/1b/Linux_Distribution_Timeline.svg

- Arquitectura Kubernetes:
    - https://kubernetes.io/es/docs/concepts/architecture/


