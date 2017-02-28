# <a name="about"></a>About

This image contains an installation Nginx 1.10.3.

Pull command:
```shell
gcloud docker -- pull launcher.gcr.io/google/nginx
```

Dockerfile for this image can be found [here](https://github.com/GoogleCloudPlatform/nginx-docker/tree/master/1.10.3).

# <a name="table-of-contents"></a>Table of Contents
* [Using Kubernetes](#using-kubernetes)
  * [Running Nginx](#running-nginx-kubernetes)
    * [Start a Nginx web server](#start-a-nginx-web-server-kubernetes)
    * [Use a persistent data volume](#use-a-persistent-data-volume-kubernetes)
  * [Web server configuration](#configuration-kubernetes)
    * [Using configuration volume](#using-configuration-volume-kubernetes)
  * [Using Nginx](#using-nginx-kubernetes)
    * [Moving the web content to Nginx](#moving-the-web-content-to-nginx-kubernetes)
    * [Testing the web server](#testing-the-web-server-kubernetes)
* [Using Docker](#using-docker)
  * [Running Nginx](#running-nginx-docker)
    * [Start a Nginx web server](#start-a-nginx-web-server-docker)
    * [Use a persistent data volume](#use-a-persistent-data-volume-docker)
  * [Web server configuration](#configuration-docker)
    * [Using configuration volume](#using-configuration-volume-docker)
  * [Using Nginx](#using-nginx-docker)
    * [Moving the web content to Nginx](#moving-the-web-content-to-nginx-docker)
    * [Testing the web server](#testing-the-web-server-docker)
* [References](#references)
  * [Ports](#references-ports)
  * [Volumes](#references-volumes)

# <a name="using-kubernetes"></a>Using Kubernetes

## <a name="running-nginx-kubernetes"></a>Running Nginx

### <a name="start-a-nginx-web-server-kubernetes"></a>Start a Nginx web server

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-nginx
  labels:
    name: some-nginx
spec:
  containers:
    - image: launcher.gcr.io/google/nginx
      name: nginx
```

Run the following to expose the ports:
```shell
kubectl expose pod some-nginx --name some-nginx-80 \
  --type LoadBalancer --port 80 --protocol TCP
kubectl expose pod some-nginx --name some-nginx-443 \
  --type LoadBalancer --port 443 --protocol TCP
```

### <a name="use-a-persistent-data-volume-kubernetes"></a>Use a persistent data volume

Web servers process and generate content. Using a persistent volume provides a way for the web server to retain the web content in the event of a container reboot or crash.

Assume /path/to/your/webcontent is the local path to the content  on your machine.

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-nginx
  labels:
    name: some-nginx
spec:
  containers:
    - image: launcher.gcr.io/google/nginx
      name: nginx
      volumeMounts:
        - name: webcontent
          mountPath: /var/www/html
  volumes:
    - name: webcontent
      persistentVolumeClaim:
        claimName: webcontent
---
# Request a persistent volume from the cluster using a Persistent Volume Claim.
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: webcontent
  annotations:
    volume.alpha.kubernetes.io/storage-class: default
spec:
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

Run the following to expose the ports:
```shell
kubectl expose pod some-nginx --name some-nginx-80 \
  --type LoadBalancer --port 80 --protocol TCP
kubectl expose pod some-nginx --name some-nginx-443 \
  --type LoadBalancer --port 443 --protocol TCP
```

## <a name="configuration-kubernetes"></a>Web server configuration

### <a name="using-configuration-volume-kubernetes"></a>Using configuration volume

Nginx can be configured through the `nginx.conf` file. Creating a configuration volume we can provide nginx with its configuration.

Assume /path/to/your/nginx.conf is the local path to the configuration file on your machine.

Create the following `configmap`:
```shell
kubectl create configmap serverconfig \
  --from-file=/path/to/your/nginx.conf
```

Copy the following content to `pod.yaml` file, and run `kubectl create -f pod.yaml`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: some-nginx
  labels:
    name: some-nginx
spec:
  containers:
    - image: launcher.gcr.io/google/nginx
      name: nginx
      volumeMounts:
        - name: serverconfig
          mountPath: /etc/nginx/nginx.conf
  volumes:
    - name: serverconfig
      configMap:
        name: serverconfig
```

Run the following to expose the ports:
```shell
kubectl expose pod some-nginx --name some-nginx-80 \
  --type LoadBalancer --port 80 --protocol TCP
kubectl expose pod some-nginx --name some-nginx-443 \
  --type LoadBalancer --port 443 --protocol TCP
```

## <a name="using-nginx-kubernetes"></a>Using Nginx

Now that Nginx is deployed you can connect to the container and test the web server.

Assume /path/to/your/webcontent is the local path to the content on your machine.

### <a name="moving-the-web-content-to-nginx-kubernetes"></a>Moving the web content to Nginx

The web server needs content to serve. Using the `kubectl` command we can move the web content to the new persistent volume we created.
```shell
kubectl cp /path/to/your/webcontent some-nginx:/var/www/html/
```

### <a name="testing-the-web-server-kubernetes"></a>Testing the web server

Using the curl command we can test that the webserver is returning the correct content.

```shell
kubectl exec -it some-nginx -- curl http://localhost
```

# <a name="using-docker"></a>Using Docker

## <a name="running-nginx-docker"></a>Running Nginx

### <a name="start-a-nginx-web-server-docker"></a>Start a Nginx web server

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  nginx:
    image: launcher.gcr.io/google/nginx
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-nginx \
  -d \
  launcher.gcr.io/google/nginx
```

### <a name="use-a-persistent-data-volume-docker"></a>Use a persistent data volume

Web servers process and generate content. Using a persistent volume provides a way for the web server to retain the web content in the event of a container reboot or crash.

Assume /path/to/your/webcontent is the local path to the content  on your machine.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  nginx:
    image: launcher.gcr.io/google/nginx
    volumes:
      - /path/to/your/webcontent:/var/www/html
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-nginx \
  -v /path/to/your/webcontent:/var/www/html \
  -d \
  launcher.gcr.io/google/nginx
```

## <a name="configuration-docker"></a>Web server configuration

### <a name="using-configuration-volume-docker"></a>Using configuration volume

Nginx can be configured through the `nginx.conf` file. Creating a configuration volume we can provide nginx with its configuration.

Assume /path/to/your/nginx.conf is the local path to the configuration file on your machine.

Use the following content for the `docker-compose.yml` file, then run `docker-compose up`.
```yaml
version: '2'
services:
  nginx:
    image: launcher.gcr.io/google/nginx
    volumes:
      - /path/to/your/nginx.conf:/etc/nginx/nginx.conf
```

Or you can use `docker run` directly:

```shell
docker run \
  --name some-nginx \
  -v /path/to/your/nginx.conf:/etc/nginx/nginx.conf \
  -d \
  launcher.gcr.io/google/nginx
```

## <a name="using-nginx-docker"></a>Using Nginx

Now that Nginx is deployed you can connect to the container and test the web server.

Assume /path/to/your/webcontent is the local path to the content on your machine.

### <a name="moving-the-web-content-to-nginx-docker"></a>Moving the web content to Nginx

The web server needs content to serve. Using the `kubectl` command we can move the web content to the new persistent volume we created.
```shell
docker cp /path/to/your/webcontent some-nginx:/var/www/html/
```

### <a name="testing-the-web-server-docker"></a>Testing the web server

Using the curl command we can test that the webserver is returning the correct content.

```shell
docker exec -it some-nginx curl http://localhost
```

# <a name="references"></a>References

## <a name="references-ports"></a>Ports

These are the ports exposed by the container image.

| **Port** | **Description** |
|:---------|:----------------|
| TCP 80 | Nginx http default port |
| TCP 443 | Nginx https secure connection over SSL |

## <a name="references-volumes"></a>Volumes

These are the filesystem paths used by the container image.

| **Path** | **Description** |
|:---------|:----------------|
| /var/www/html | Volume location where web server will parse and serve static and dynamic content. |
| /etc/nginx/nginx.conf | Volume location where web server is configured. |