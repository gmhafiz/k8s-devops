```sh
#kubectl apply -f secret.yaml # use SealedSecret instead
kubectl apply -f configmap.yaml
kubectl apply -f pv.yaml
kubectl apply -f pvc.yaml
```

Install SealedSecret

```sh
mkdir -p ~/Downloads/kubeseal
cd ~/Downloads/kubeseal
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.22.0/kubeseal-0.22.0-darwin-amd64.tar.gz
tar -xvzf kubeseal-0.22.0-darwin-amd64.tar.gz
sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

Install SealedSecret controller to k8s

```sh
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.22.0/controller.yaml
```

Create SealedSecret object from our `secret.yaml`

```sh
cat secrets.yaml | kubeseal \
--controller-namespace kube-system \
--controller-name sealed-secrets-controller \
--format yaml \
> sealed-secrets.yaml
```

Apply SealedSecret along with the rest of yaml files

```sh
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

```sh
alias k=kubectl
k get po
```

```sh
k logs -f postgres-7c6b976c95-wjls2
```

```
PostgreSQL Database directory appears to contain a database; Skipping initialization

2023-05-22 02:45:53.640 UTC [1] LOG:  starting PostgreSQL 15.1 (Debian 15.1-1.pgdg110+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 10.2.1-6) 10.2.1 20210110, 64-bit
2023-05-22 02:45:53.641 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
2023-05-22 02:45:53.641 UTC [1] LOG:  listening on IPv6 address "::", port 5432
2023-05-22 02:45:53.646 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
2023-05-22 02:45:53.653 UTC [27] LOG:  database system was shut down at 2023-05-22 02:40:23 UTC
2023-05-22 02:45:53.659 UTC [1] LOG:  database system is ready to accept connections
```

Connect

```sh
export POSTGRES_PASSWORD=$(kubectl get secret postgres-secret-config -o jsonpath="{.data.POSTGRES_PASSWORD}" | base64 -d)

kubectl run psql-client --rm --tty -i --restart='Never' \
  --namespace default \
  --image postgres:16.0 \
  --env="PGPASSWORD=$POSTGRES_PASSWORD" \
  --command -- psql --host postgres -U user -d my_db -p 5432
```

Quit psql-client

```
\q
```

Port Forward to port 45432 in your computer

```sh
kubectl port-forward --namespace default svc/postgres 45432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U app1 -d app_db -p 5432
```

Delete (in order)

```sh
k delete svc postgres
k delete deploy postgres
k delete pvc postgres-pv-claim
k delete pv postgres-pv-volume
k delete cm db-credentials
```


More info at blog https://www.gmhafiz.com/blog/deploy-applications-in-kubernetes/#database