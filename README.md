# Deploy Three Applications in Kubernetes Demo

Full guide at https://www.gmhafiz.com/blog/deploy-applications-in-kubernetes/


## 1. Database

Instructions are in this file.

```shell
vim ./db/README.md
```

## API

### Dockerise

```sh
cd ..
git clone https://github.com/gmhafiz/k8s-api
cd k8s-api

cat README.md
```

### Deploy

Set image tag

```sh
TAG=$(git rev-parse HEAD)
echo $TAG
```

### Build image

```sh
docker build -f Dockerfile-api -t gmhafiz/api:$TAG .
docker build -f Dockerfile-migrate -t gmhafiz/migrate:$TAG .
```

### Push to container registry

```sh
docker push gmhafiz/api:${TAG}
docker push gmhafiz/migrate:${TAG}
```

### 2. Deploy API

```sh
kubectl apply -f configmap.yaml
kubectl apply -f server.yaml
kubectl apply -f service.yaml
```

Edit migrate image

```vim
vim migrate-job.yaml
```

Run migration

```sh
kubectl apply -f migrate-job.yaml
```
View migrate log

```sh
kubectl get pods
kubectl logs -f migrate-job-zhmpt
```

```
# returns

2023/09/09 02:49:30 starting migrate...
2023/09/09 02:49:30 reading env
2023/09/09 02:49:30 connecting to database... 
2023/09/09 02:49:30 database connected
2023/09/09 02:49:30 OK   20230302080119_create_randoms_table.sql (42.53ms)
2023/09/09 02:49:30 goose: no migrations to run. current version: 20230302080119
2023/09/09 02:49:30 goose: version 20230302080119
```

### Port Forward

Port forward

```sh
kubectl port-forward --namespace default svc/server 3080:3080
```


### Test

In a new terminal

```sh
curl http://localhost:3080/randoms
```


## 3. Frontend

### Build

```sh
cd ..
git clone https://github.com/gmhafiz/k8s-web
cd k8s-web

TAG=$(git rev-parse HEAD)
echo $TAG
docker build -t gmhafiz/web:${TAG} .
docker push gmhafiz/web:${TAG} 
```

### Deploy

Change directory

```sh
cd k8s-devops/web
```


Edit image tag

```sh
vim web.yaml
# gmhafiz/web:b169cbaecb549b11f5597ef6dd2f7647fd1133cc <--
```

```sh
kubectl apply -f web.yaml
```

### Port forward

```sh
kubectl port-forward svc/web 8080:8080
```

Go to http://localhost:8080 and enjoy!
