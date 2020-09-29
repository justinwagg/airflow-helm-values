# airflow-helm-values

These are my custom Helm values for deploying Airflow 1.10.12 on my TuringPi cluster

## Configuration

The original [values.yaml](https://github.com/helm/charts/blob/master/stable/airflow/values.yaml) from the Helm stable/airflow repo is a good reference when customizing.

The Airflow image is a build I created specifically for arm which can be found [here](https://hub.docker.com/repository/docker/justinmwagg/airflow/general).

The chart also relies on an arm built version of `alpine-docker/git` which can be found [here](https://github.com/alpine-docker/git).

Luckily there was already a multi-arch build of `k8s.gcr.io/git-sync/git-sync` to rely on.

- For my log volume I'm using an NFS on my local network. You might want to configure something different in `airflow-logs-pv.yaml`.

- I'm also using an external postgres database for the backend. You can run a db easily within the cluster. The original values.yaml has the example.

## Setup

While on the master node of the cluster create a namespace for our Airflow deployment.

```bash
sudo kubectl create namespace airflow
```

Add secrets to the namespace

```bash
sudo kubectl create secret generic airflow-fernet-key \
    --from-literal=value=<YOUR_KEY> \
    -n airflow

sudo kubectl create secret generic postgresql-password \
    --from-literal=password=<YOUR_PASSWORD> \
    -n airflow

sudo kubectl create secret generic git-sync-ssh-key \
    --from-file=id_rsa.pub=/root/.ssh/id_rsa.pub \
    --from-file=id_rsa=/root/.ssh/id_rsa \
    --from-file=gitSshKey=/root/.ssh/id_rsa \
    --from-file=known_hosts=/root/.ssh/known_hosts \
    -n airflow
```

Apply services

```bash
sudo kubectl apply -f webserver-service.yaml -n airflow
sudo kubectl apply -f airflow-logs-pv.yaml -n airflow
```

Install the helm chart 

```bash
helm install airflow stable/airflow --namespace "airflow" --values ./custom-values.yaml
```

you should get a message telling you `Congratulations. You have just deployed Apache Airflow!`. 

A this point you should be able to get the Web UI port by running 

```bash
sudo kubectl get svc -n airflow -o wide
```

and grabbing the `service/airflow-webserver` PORT value and going to `http://<YOUR_MASTER_IP>:<PORT>`.
