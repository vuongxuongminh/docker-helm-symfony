# Modern containerize template for Symfony

![Build Status](https://github.com/vuongxuongminh/docker-helm-symfony/workflows/Docker/badge.svg)

## About it

This is a clean template base on `symfony/skeleton` (Symfony 5) with common Symfony packages:

+ Debug (dev env).
+ Mailer.
+ Messenger.
+ ORM.

And set of Docker services, include basic Helm chart help you save time and easy to deploy it in your Kubernetes cluster.

## Prerequisites

+ Helm version 3.0+
+ Docker compose 1.17+
+ Docker engine 17.09+
+ Kubernetes cluster 1.14+

## Getting started

Start by downloading [distribution .tar.gz file](https://github.com/vuongxuongminh/docker-helm-symfony/releases), or generate a GitHub repo from this repo. 

After download and extract, open a terminal, and navigate to the directory containing your project. Run the following command to start all services using Docker Compose:

```shell script
$ docker-compose pull # Download the latest versions of the pre-built images
$ docker-compose up -d # Running in detached mode
```

This starts the following services:

| Name          |           Description                                               | Ports | Environments |
|---------------|---------------------------------------------------------------------|------ |--------------|
| fpm           | FastCGI process manager with PHP-FPM 7.4.12, Composer.              | n/a   | all          |
| supervisor    | Process control system with PHP 7.4.12, Supervisor 4.1.0, Composer  | 9000  | all          |
| nginx         | Reverse proxy handle request with NGINX 1.17                        | 80    | all          |
| setup         | Setup service help run migration & install Composer package         | n/a   | dev          |
| promtail      | Ship logs of Nginx & FPM & Supervisor services to Loki              | n/a   | dev          |
| loki          | Logs datasource                                                     | n/a   | dev          |
| grafana       | Logs visualization                                                  | 3000  | dev          |
| rabbitmq      | Message broker                                                      | 15672 | dev          |
| mysql         | Mysql database server                                               | 3306  | dev          |
| mailhog       | Mail server mock                                                    | 8025  | dev          |

This results in the following running containers:

```shell script
$ docker-compose ps

              Name                            Command                  State                                          Ports                                    
---------------------------------------------------------------------------------------------------------------------------------------------------------------
docker-helm-symfony_fpm_1          docker-entrypoint fpm            Up (healthy)   9000/tcp                                                                    
docker-helm-symfony_grafana_1      /run.sh                          Up             0.0.0.0:3000->3000/tcp                                                      
docker-helm-symfony_loki_1         /usr/bin/loki -config.file ...   Up             3100/tcp                                                                    
docker-helm-symfony_mailhog_1      MailHog                          Up             1025/tcp, 0.0.0.0:8025->8025/tcp                                            
docker-helm-symfony_mysql_1        docker-entrypoint.sh --def ...   Up             3306/tcp, 33060/tcp                                                         
docker-helm-symfony_nginx_1        docker-entrypoint nginx -g ...   Up (healthy)   0.0.0.0:80->80/tcp                                                          
docker-helm-symfony_promtail_1     /usr/bin/promtail -config. ...   Up                                                                                         
docker-helm-symfony_rabbitmq_1     docker-entrypoint.sh rabbi ...   Up             15671/tcp, 0.0.0.0:15672->15672/tcp, 25672/tcp, 4369/tcp, 5671/tcp, 5672/tcp
docker-helm-symfony_setup_1        docker-entrypoint setup          Exit 0                                                                                     
docker-helm-symfony_supervisor_1   docker-entrypoint supervisor     Up (healthy)   0.0.0.0:9000->9000/tcp         
```

If you want to change a PHP or Nginx version, open `docker-compose.yaml` and add build args:

+ `PHP_VERSION` for change php version (ex: `PHP_VERSION: '7.3'`).
+ `NGINX_VERSION` for change nginx version (ex: `NGINX_VERSION: '1.16'`).

Open [Dockerfile](/docker/Dockerfile) to see more build args.

Now you can visiting:

+ Your Symfony app: http://localhost
+ Supervisor: http://localhost:9000 (Username: `root`, Password: `root` you can set it in `docker-compose.override.yaml`)
+ RabbitMQ: http://localhost:15672 (Username: `guest`, Password: `guest` you can set it in `docker-compose.override.yaml`)
+ Grafana: http://localhost:3000/dashboards (Username: `admin`, Password: `admin` you can set it in `docker-compose.override.yaml`)
+ Mailhog: http://localhost:8025

And access MySQL via port 3306 (Username: `root`, Password: `root` you can set it in `docker-compose.override.yaml`)

### Grafana dashboards

+ Logs visualization

![Logs visualization](./screenshots/dashboard.png)

## Deploy project into Kubernetes cluster

### Install

Firstly you need to update helm dependencies by running:

```shell script
$ helm dep update ./helm/symfony
```

And install chart:

```shell script
$ helm install app ./helm/symfony
```

The command deploys your project on the Kubernetes cluster in the default configuration. The [Parameters](#parameters) section lists the parameters that can be configured during installation.

### Uninstalling the Chart

To uninstall/delete the app deployment:

```shell script
$ helm uninstall app
```

### Parameters

The following table lists the configurable parameters of this chart and their default values.

| Parameter                                    | Description                                                                                                                            | Default                                                                                   |
|----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------|
| imagePullSecrets                             | Docker registry secret names as an array.                                                                                              | `[]`                                                                                      |
| imagePullPolicy                              | Image pull policy.                                                                                                                     | `Always`                                                                                  |
| php.image.repository                         | PHP image name.                                                                                                                        | `vuongxuongminh/docker-helm-symfony-php`                                                  |
| php.image.tag                                | PHP image tag.                                                                                                                         | `prod`                                                                                    |
| php.supervisor.basicAuth.username            | Supervisor basic auth username.                                                                                                        | `user`                                                                                    |
| php.supervisor.basicAuth.password            | Supervisor basic auth password.                                                                                                        | `!ChangeMe!`                                                                              |
| php.supervisor.replicaCount                  | Supervisor replica count.                                                                                                              | `1`                                                                                       |
| php.supervisor.nodeSelector                  | Supervisor node labels for pod assignment.                                                                                             | `{}`                                                                                      |
| php.supervisor.tolerations                   | Supervisor toleration labels for pod assignment.                                                                                       | `[]`                                                                                      |
| php.supervisor.affinity                      | Supervisor affinity settings for pod assignment.                                                                                       | `{}`                                                                                      |
| php.supervisor.podAnnotations                | Supervisor pod annotations.                                                                                                            | `{}`                                                                                      |
| php.supervisor.resources                     | Supervisor resource needs and limits to apply to the pod.                                                                              | `{}`                                                                                      |
| php.fpm.replicaCount                         | FPM replica count.                                                                                                                     | `1`                                                                                       |
| php.fpm.nodeSelector                         | FPM node labels for pod assignment.                                                                                                    | `{}`                                                                                      |
| php.fpm.tolerations                          | FPM toleration labels for pod assignment.                                                                                              | `[]`                                                                                      |
| php.fpm.affinity                             | FPM affinity settings for pod assignment.                                                                                              | `{}`                                                                                      |
| php.fpm.podAnnotations                       | FPM pod annotations.                                                                                                                   | `{}`                                                                                      |
| php.fpm.resources                            | FPM resource needs and limits to apply to the pod.                                                                                     | `{}`                                                                                      |
| php.env                                      | Use to set `APP_ENV` env in Supervisor & FPM containers.                                                                               | `prod`                                                                                    |
| php.debug                                    | Use to set `APP_DEBUG` env in Supervisor & FPM containers.                                                                             | `0`                                                                                       |
| php.trustedHosts                             | Use to set `TRUSTED_HOSTS` env in Supervisor & FPM containers.                                                                         | `^.*\\.example\\.com$`                                                                    |
| php.trustedProxies                           | Use to set `TRUSTED_PROXIES` env in Supervisor & FPM containers.                                                                       | `[10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16]` list common CIDR default of K8S.            |
| nginx.image.repository                       | Nginx image name.                                                                                                                      | `vuongxuongminh/docker-helm-symfony-nginx`                                                |
| nginx.image.tag                              | Nginx image tag.                                                                                                                       | `prod`                                                                                    |
| nginx.replicaCount                           | Nginx replica count.                                                                                                                   | `1`                                                                                       |
| nginx.nodeSelector                           | Nginx node labels for pod assignment.                                                                                                  | `{}`                                                                                      |
| nginx.tolerations                            | Nginx toleration labels for pod assignment.                                                                                            | `[]`                                                                                      |
| nginx.affinity                               | Nginx affinity settings for pod assignment.                                                                                            | `{}`                                                                                      |
| nginx.podAnnotations                         | Nginx pod annotations.                                                                                                                 | `{}`                                                                                      |
| nginx.resources                              | Nginx resource needs and limits to apply to the pod.                                                                                   | `{}`                                                                                      |
| ingress.annotations                          | Ingress annotations.                                                                                                                   | `{}`                                                                                      |
| ingress.tls                                  | Enable ingress with tls.                                                                                                               | `false`                                                                                   |
| ingress.nginx.host                           | Ingress Nginx host name.                                                                                                               | `nginx.example`                                                                           |
| ingress.nginx.tlsSecret                      | Ingress Nginx tls secret name.                                                                                                         | `nginx-tls`                                                                               |
| ingress.supervisor.host                      | Ingress Supervisor host name.                                                                                                          | `supervisor.example`                                                                      |
| ingress.supervisor.tlsSecret                 | Ingress Supervisor tls secret name.                                                                                                    | `supervisor-tls`                                                                          |
| mysql.internal                               | Use internal MySQL (sub chart) or not.                                                                                                 | `true`                                                                                    |
| mysql.fullnameOverride                       | MySQL internal fullname.                                                                                                               | `sub-chart-mysql`                                                                         |
| mysql.mysqlDatabase                          | MySQL internal database name.                                                                                                          | `test`                                                                                    |
| mysql.mysqlUser                              | MySQL internal username.                                                                                                               | `user`                                                                                    |
| mysql.mysqlPassword                          | MySQL internal password.                                                                                                               | `!ChangeMe!`                                                                              |
| mysql.persistence.enabled                    | MySQL internal use a PVC to persist data.                                                                                              | `false`                                                                                   |
| mysql.nodeSelector                           | MySQL internal node labels for pod assignment.                                                                                         | `{}`                                                                                      |
| mysql.tolerations                            | MySQL internal toleration labels for pod assignment.                                                                                   | `[]`                                                                                      |
| mysql.podAnnotations                         | MySQL internal pod annotations.                                                                                                        | `{}`                                                                                      |
| mysql.resources                              | MySQL internal resource needs and limits to apply to the pod.                                                                          | `{}`                                                                                      |
| mysql.externalDsn                            | MySQL externalDsn use to set `DATABASE_URL` env in Supervisor & FPM containers. Use when `mysql.internal` is false.                    | `''`                                                                                      |
| rabbitmq.internal                            | Use internal RabbitMQ (sub chart) or not.                                                                                              | `true`                                                                                    |
| rabbitmq.fullnameOverride                    | RabbitMQ internal fullname.                                                                                                            | `sub-chart-rabbitmq`                                                                      |
| rabbitmq.updateStrategy.type                 | RabbitMQ internal statefulset update strategy policy.                                                                                  | `OnDelete`                                                                                |
| rabbitmq.persistence.enabled                 | RabbitMQ internal use a PVC to persist data.                                                                                           | `false`                                                                                   |
| rabbitmq.service.port                        | RabbitMQ internal service port.                                                                                                        | `5672`                                                                                    |
| rabbitmq.rabbitmq.username                   | RabbitMQ internal username.                                                                                                            | `user`                                                                                    |
| rabbitmq.rabbitmq.password                   | RabbitMQ internal password.                                                                                                            | `!ChangeMe!`                                                                              |
| rabbitmq.ingress.enabled                     | RabbitMQ internal enable ingress for management site or not.                                                                           | `true`                                                                                    |
| rabbitmq.ingress.tls                         | RabbitMQ internal enable ingress with tls.                                                                                             | `false`                                                                                   |
| rabbitmq.ingress.tlsSecret                   | RabbitMQ internal ingress tls secret name.                                                                                             | `rabbitmq-tls`                                                                            |
| rabbitmq.ingress.hostName                    | RabbitMQ internal ingress host name.                                                                                                   | `rabbitmq.example`                                                                        |
| rabbitmq.ingress.annotations                 | RabbitMQ internal ingress annotations.                                                                                                 | `{}`                                                                                      |
| rabbitmq.nodeSelector                        | RabbitMQ internal node labels for pod assignment.                                                                                      | `{}`                                                                                      |
| rabbitmq.tolerations                         | RabbitMQ internal toleration labels for pod assignment.                                                                                | `[]`                                                                                      |
| rabbitmq.affinity                            | RabbitMQ internal affinity settings for pod assignment.                                                                                | `{}`                                                                                      |
| rabbitmq.podAnnotations                      | RabbitMQ internal pod annotations.                                                                                                     | `{}`                                                                                      |
| rabbitmq.resources                           | RabbitMQ internal resource needs and limits to apply to the pod.                                                                       | `{}`                                                                                      |
| rabbitmq.externalDsn                         | RabbitMQ externalDsn use to set `MESSENGER_TRANSPORT_DSN` env in Supervisor & FPM containers. Use when `rabbitmq.internal` is false.   | `''`                                                                                      |

Specify each parameter using the --set key=value[,key=value] argument to helm install. For example,

```shell script
$ helm install app ./helm/symfony \
  --set ingress.nginx.hostName=nginx.example.com,ingress.supervisor.hostName=supervisor.example.com,rabbitmq.ingress.hostName=rabbitmq.example.com
```

The above command sets the Nginx host name to `nginx.example.com`, Supervisor host name to `supervisor.example.com` and RabbitMQ management host name to `rabbitmq.example.com`.

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example,

```shell script
$ helm install app ./helm/symfony -f values.yaml
```

> Tip: You can use the default [values.yaml](/helm/symfony/values.yaml)


### About database migration job

When you install or upgrade chart, Helm automatically invoke [database migration job](/helm/symfony/templates/hook/job.yaml) after it done thanks to [chart hook](https://helm.sh/docs/topics/charts_hooks/).
