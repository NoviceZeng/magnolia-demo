# CMS Demo

This is a demo for deploy Magnolia CMS by helm chart in k8s installed by minikube.

# Configure minikube and k8s
1. Start k8s as below screenshot, for connect the cluster locally and CICD purpose, we set `--listen-address=0.0.0.0`, but for security reasones, we`d better to **configure a security group or firewall rules**.
<img width="937" alt="image" src="https://github.com/user-attachments/assets/85e340e6-f91b-493d-b399-49d65771e610" />

2. Check port mapping of api server of 8443, we can see that mapping port is `32771`, and we can use it.
```
docker port minikube
22/tcp -> 0.0.0.0:32768
2376/tcp -> 0.0.0.0:32769
5000/tcp -> 0.0.0.0:32770
8443/tcp -> 0.0.0.0:32771
32443/tcp -> 0.0.0.0:32772
```

3. Copy ca, crt, key to your local machine.  
```
scp root@EXTERNAL_IP:~/.minikube/ca.crt ~/.kube/ca.
scp root@EXTERNAL_IP:~/.minikube/profiles/minikube/client.crt  ~/.kube/client.crt
scp root@EXTERNAL_IP:~/.minikube/profiles/minikube/client.key ~/.kube/client.key
```

4. Create kubeconfig file in your local machine
```
cat ~/.kube/config 
apiVersion: v1
clusters:
- cluster:
    certificate-authority: ca.crt
    server: https://EXTERNAL_IP:32771
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: client.crt
    client-key: client.key
```


# Deploy CMS
It is better to deploy by a CICD tools like Github workflow, Gitlab runner or Jenkins, but for test purpose, we use helm chart to manually deploy it.
Use `helm install  mag magnoliachart  --dry-run --debug` for dry run first.  
Use `helm install  mag magnoliachart  --dry-run --debug` to deploy directly.
```
root@iZj6ch0w241h6gs5syc4lsZ:~# helm list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
magnolia-demo   default         1               2024-12-29 17:14:29.99841 +0800 CST     deployed        magnolia-0.1.0  6.1.1      
root@iZj6ch0w241h6gs5syc4lsZ:~# kubectl  get ingress,svc,pods
NAME                                      CLASS    HOSTS                                             ADDRESS   PORTS   AGE
ingress.networking.k8s.io/magnolia-demo   <none>   author.magnoliademo.com,public.magnoliademo.com             80      3m43s

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes             ClusterIP   10.96.0.1      <none>        443/TCP   28m
service/magnolia-demo-author   ClusterIP   10.109.43.63   <none>        80/TCP    3m43s
service/magnolia-demo-public   ClusterIP   10.104.6.77    <none>        80/TCP    3m43s

NAME                         READY   STATUS    RESTARTS   AGE
pod/magnolia-demo-author-0   1/1     Running   0          3m43s
pod/magnolia-demo-public-0   1/1     Running   0          3m43s
root@iZj6ch0w241h6gs5syc4lsZ:~# 
root@iZj6ch0w241h6gs5syc4lsZ:~# kubectl  get ingress,svc,pods
NAME                                      CLASS    HOSTS                                             ADDRESS   PORTS   AGE
ingress.networking.k8s.io/magnolia-demo   <none>   author.magnoliademo.com,public.magnoliademo.com             80      12m

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/kubernetes             ClusterIP   10.96.0.1      <none>        443/TCP   37m
service/magnolia-demo-author   ClusterIP   10.109.43.63   <none>        80/TCP    12m
service/magnolia-demo-public   ClusterIP   10.104.6.77    <none>        80/TCP    12m

NAME                         READY   STATUS    RESTARTS   AGE
pod/magnolia-demo-author-0   1/1     Running   0          12m
pod/magnolia-demo-public-0   1/1     Running   0          12m
```

# Show
Before we login, we need make `port-forward`
```
kubectl port-forward svc/magnolia-demo-author  8000:80
kubectl port-forward svc/magnolia-demo-public  8001:80
```
Then we login by superuser/superuser
## author
<img width="1509" alt="image" src="https://github.com/user-attachments/assets/9e0d6ed7-85cf-4cb2-a55c-7d1a36a204bb" />
## public
<img width="1503" alt="image" src="https://github.com/user-attachments/assets/578c5b84-89c7-4899-802c-d2f479df8f70" />

# CICD

Refer to the [pipeline](.github/workflows/deploy_cms.yml).


