### Deploying PHP Guestbook application with Redis
```
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-master-deployment.yaml
kubectl get pods
kubectl logs -f <POD_NAME>

kubectl apply -f https://k8s.io/examples/application/guestbook/redis-master-service.yaml
kubectl get services

kubectl apply -f https://k8s.io/examples/application/guestbook/redis-slave-deployment.yaml
kubectl get pods

kubectl apply -f https://k8s.io/examples/application/guestbook/redis-slave-service.yaml
kubectl get services

kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
kubectl get pods -l app=guestbook -l tier=frontend

kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
kubectl get services

minikube service frontend --url
kubectl get service frontend

kubectl scale deployment frontend --replicas=5
kubectl get pods

kubectl delete deployment -l app=redis
kubectl delete service -l app=redis
kubectl delete deployment -l app=guestbook
kubectl delete service -l app=guestbook
```
