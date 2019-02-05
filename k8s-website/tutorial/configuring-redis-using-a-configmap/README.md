```shell
kubectl create configmap example-redis-config --from-file=pods/config/redis-config
kubectl get configmap example-redis-config -o yaml
kubectl create -f https://k8s.io/examples/pods/config/redis-pod.yaml
```

```
$ kubectl exec -it redis redis-cli127.0.0.1:6379> CONFIG GET maxmemory
1) "maxmemory"
2) "2097152"
$ 127.0.0.1:6379> CONFIG GET maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
127.0.0.1:6379>
```
