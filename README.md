# Deployment of Go Web Application

## Deployment of application using manifest file

Let's start with writing a Multistage Dockerfile

```cli
docker build -t rivadias/go-web-app:v1 .
docker run -p 8080:8080 -it rivadias/go-web-app:v1
docker push rivadias/go-web-app:v1
```

Create a k8s/manifests: deployment.yaml, service.yaml, ingress.yaml

```cli
 aws configure
```

```cli
 eksctl create cluster --name demo-cluster --region us-east-1
```
```cli
kubectl apply -f k8s/manifests/deployment.yaml

kubectl apply -f k8s/manifests/service.yaml

kubectl apply -f k8s/manifests/ingress.yaml
```

Edit the service and change it to NodePort 
```cli
kubectl edit svc nameofservice
```
```cli
kubectl get nodes -o wide
```
![Screenshot 2025-05-31 124630](https://github.com/user-attachments/assets/5e7ad1ff-672f-4182-9c3d-6c63cc37b916)

Copy external_ip of the node and open new tab- ip:service_port/courses

Create a ingress controller to watch over ingress
```cli
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
```

```cli
kubectl get pods -n ingress-nginx

kubectl get ing
```

Since the service runs on the host: go-web-app.local lets do the DNS mapping 
```cli
nslookup lb_address

sudo vim /etc/hosts
```
in the host paste the address- ip_address  go-web-app.local

New-tab: go-web-app.local/home

![Screenshot 2025-05-31 154054](https://github.com/user-attachments/assets/2905b1e4-812e-4bff-9e41-a6e41ce2f20f)


## Deployment using helm


Create a helm folder

```cli
helm create go-web-app-chart
cd go-web-app-chart

```
Move the earlier manifest file to templates and update the values.yaml file

```cli
helm install go-web-app ./go-web-app-chart
```

```cli
kubectl get deployment
kubectl get svc
```
![Screenshot 2025-05-31 163051](https://github.com/user-attachments/assets/471119ae-1130-4298-ab3f-7c3add6f28e2)


# Deployment using CI/CD pipeline, CI- GithubActions, CD- ArgoCD


Create a .GitHub/workflows/ci.yaml

Create a ci pipeline, also add secrets to the GitHub repo for Dockerhub user, password and GitHub token

![Screenshot 2025-05-31 191459](https://github.com/user-attachments/assets/b6d61929-413d-4930-8b54-f64d28f2fcbf)

```cli
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
```cli
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

```cli
kubectl get svc -n argocd
```
open on browser with the address

for ArgoCD password

```cli
kubectl get secrets -n argocd
kubectl edit secret argocd-initial-admin-secret -n argocd
echo passwrd_string | base64 --decode
```

Add the application to ArgoCD
![Screenshot 2025-05-31 194120](https://github.com/user-attachments/assets/82b2facd-55fd-47e0-8b84-67ca36bf14a4)


```cli
kubectl get deployment
kubectl get svc
kubectl get ing
```

New tab: go-web-app.local/cources

![Screenshot 2025-05-31 194144](https://github.com/user-attachments/assets/a751b8eb-7e56-4ff9-a2a0-c151c6dd71b4)




