# app-test
App-rotoro

Подготавливаем хосты тут - https://github.com/joos-virt/k8s-install-work/

**Rotoro**
```
kubectl run nginx --image=nginx - создаем
kubectl get pods - смотрим
kubectl describe pod nginx - подробная информация
kubectl get pods -o wide - показывает дополнительные поля
kubectl delete pod nginx
```
**pod.yml**
```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    type: frontend
spec:
  containers:
  - name: nginx
    image: nginx
```
```
kubectl create -f pod.yml - создаем
kubectl get pods - смотрим
kubectl describe pod nginx - подробная информация
```
**replicasets.yml**
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp
spec:
  template:
    metadata:
      name: nginx2
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      app: myapp
```
```
kubectl create -f replicasets.yml - создаем
kubectl get rs - смотрим replicasets
kubectl get po - смотрим поды
kubectl delete po myapp-replicaset-28w4l - удаляем под
kubectl get po - проверяем что автоматом поднялся еще один
kubectl edit rs myapp-replicaset - редактируем rs через vi
kubectl scale rs myapp-replicaset --replicas=2 - или редактор реплик на лету
kubectl delete rs myapp-replicaset
```
**deployment.yml**
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: nginx
    tier: frontend
spec:
  selector:
    matchLabels:
      app: myapp
  replicas: 4
  template:
    metadata:
      name: nginx2
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
```
```
kubectl create -f deployment.yml - создаем
kubectl get deploy - смотрим деплои
kubectl get po - смотрим поды
kubectl describe deployment myapp-deployment - описание
kubectl get all - выводит все
```
**Rollouts**
```
kubectl rollout status deploy myapp-deployment - состояние выкатки
kubectl rollout history deploy myapp-deployment - просмотр списка ревизий и истории изменения
kubectl create -f deployment.yml --record - запись команды в hidtory
kubectl edit deploy myapp-deployment - редактируем, меняем версию nginx контейнера
kubectl rollout status deploy myapp-deployment - смотрим что происходит
kubectl describe deployment myapp-deployment - смотрим события
kubectl set image deployment myapp-deployment nginx=nginx:1.24.0-perl --record - меняем версию через строку
kubectl rollout undo deployment/myapp-deployment - откат
```
**service.yml**
```yml
apiVersion: v1
kind: Service
metadata:
    name: myapp-service
spec:
    type: NodePort
    ports:
      - port: 80
        targetPort: 80
        nodePort: 30004
    selector:
      app: myapp
```
```
kubectl create -f service.yml - создаем
kubectl get svc - смотрим сервисы
minikube service myapp-service --url - получаем внешний линк
ip:30004 - или идем на ноду и смотрим по порту
```
**App**

voting-app-pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
    name: voting-app-pod
    labels:
        name: voting-app-pod
        app: demo-voting-app
spec:
    containers:
        - name: voting-app
          image: dockersamples/examplevotingapp_vote
          ports:
            - containerPort: 80
```
result-app-pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
    name: result-app-pod
    labels:
        name: result-app-pod
        app: demo-voting-app
spec:
    containers:
        - name: result-app
          image: dockersamples/examplevotingapp_result
          ports:
            - containerPort: 80

```
postgres-pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
    name: postgres-pod
    labels:
        name: postgres-pod
        app: demo-voting-app
spec:
    containers:
        - name: postgres
          image: postgres
          ports:
              - containerPort: 5432
          env:
              - name: POSTGRES_USER
                value: "postgres"
              - name: POSTGRES_PASSWORD
                value: "postgres"
              - name: POSTGRES_HOST_AUTH_METHOD
                value: trust
```
redis-pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
    name: redis-pod
    labels:
        name: redis-pod
        app: demo-voting-app
spec:
    containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
```
worker-app-pod.yml
```yml
apiVersion: v1
kind: Pod
metadata:
    name: worker-app-pod
    labels:
        name: worker-app-pod
        app: demo-voting-app
spec:
    containers:
        - name: voting-app
          image: dockersamples/examplevotingapp_worker
```
**Service**

postgres-service.yml
```yml
apiVersion: v1
kind: Service
metadata:
    name: db
    labels:
        name: postgres-service
        app: demo-voting-app
spec:
    ports:
        - port: 5432
          targetPort: 5432
    selector:
        name: postgres-pod
        app: demo-voting-app
```
redis-service.yml
```yml
apiVersion: v1
kind: Service
metadata:
    name: redis
    labels:
        name: redis-service
        app: demo-voting-app
spec:
    ports:
        - port: 6379
          targetPort: 6379
    selector:
        name: redis-pod
        app: demo-voting-app
```
result-service.yml
```yml
apiVersion: v1
kind: Service
metadata:
    name: result-service
    labels:
        name: result-service
        app: demo-voting-app
spec:
    type: NodePort
    ports:
        - port: 80
          targetPort: 80
          nodePort: 30005
    selector:
        name: result-app-pod
        app: demo-voting-app
```
voting-app-service.yml
```yml
apiVersion: v1
kind: Service
metadata:
    name: voting-service
    labels:
        name: voting-service
        app: demo-voting-app
spec:
    type: NodePort
    ports:
        - port: 80
          targetPort: 80
          nodePort: 30004
    selector:
        name: voting-app-pod
        app: demo-voting-app
```
**Start**
```
kubectl create -f voting-app-pod.yml
kubectl create -f voting-app-service.yml
kubectl create -f redis-pod.yml
kubectl create -f redis-service.yml
kubectl create -f postgres-pod.yml
kubectl create -f postgres-service.yml
kubectl create -f worker-app-pod.yml
kubectl create -f result-app-pod.yml
kubectl create -f result-service.yml
kubectl get po,svc
ip:30004
ip:30005
```

**App + Deployments**

postgres-deploy.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: postgres-deploy
    labels:
      name: postgres-deploy
      app: demo-voting-app
spec:
    replicas: 2
    selector:
      matchLabels:
        name: postgres-pod
        app: demo-voting-app
    template:
      metadata:
          name: postgres-pod
          labels:
              name: postgres-pod
              app: demo-voting-app
      spec:
          containers:
              - name: postgres
                image: postgres
                ports:
                    - containerPort: 5432
                env:
                    - name: POSTGRES_USER
                      value: "postgres"
                    - name: POSTGRES_PASSWORD
                      value: "postgres"
                    - name: POSTGRES_HOST_AUTH_METHOD
                      value: trust
```
redis-deploy.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: redis-deploy
    labels:
        name: redis-deploy
        app: demo-voting-app
spec:
    replicas: 2
    selector:
        matchLabels:
          name: redis-pod
          app: demo-voting-app
    template:
      metadata:
          name: redis-pod
          labels:
              name: redis-pod
              app: demo-voting-app
      spec:
          containers:
              - name: redis
                image: redis
                ports:
                  - containerPort: 6379
```
result-deploy.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: result-deploy
    labels:
        name: result-deploy
        app: demo-voting-app
spec:
    replicas: 2
    selector:
        matchLabels:
          name: result-app-pod
          app: demo-voting-app
    template:
      metadata:
          name: result-app-pod
          labels:
              name: result-app-pod
              app: demo-voting-app
      spec:
          containers:
              - name: result-app
                image: dockersamples/examplevotingapp_result
                ports:
                  - containerPort: 80
```
voting-app-deploy.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: voting-app-deploy
    labels:
        name: voting-app-deploy
        app: demo-voting-app
spec:
    replicas: 2
    selector:
        matchLabels:
          name: voting-app-pod
          app: demo-voting-app
    template:
      metadata:
          name: voting-app-pod
          labels:
              name: voting-app-pod
              app: demo-voting-app
      spec:
          containers:
              - name: voting-app
                image: dockersamples/examplevotingapp_vote
                ports:
                  - containerPort: 80
```
worker-deploy.yml
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: worker-deploy
    labels:
      name: worker-deploy
      app: demo-voting-app
spec:
    replicas: 2
    selector:
        matchLabels:
          name: worker-app-pod
          app: demo-voting-app
    template:
      metadata:
          name: worker-app-pod
          labels:
             name: worker-app-pod
             app: demo-voting-app
      spec:
          containers:
              - name: voting-app
                image: dockersamples/examplevotingapp_worker
```
**Start**
```
kubectl create -f voting-app-deploy.yml
kubectl create -f voting-app-service.yml
kubectl create -f redis-deploy.yml
kubectl create -f redis-service.yml
kubectl create -f postgres-deploy.yml
kubectl create -f postgres-service.yml
kubectl create -f worker-deploy.yml
kubectl create -f result-deploy.yml
kubectl create -f result-service.yml
```
