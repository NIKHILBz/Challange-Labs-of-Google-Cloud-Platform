## Let's start with defining some variables given by Cloud Skill Boosts
```
export COLOUR=
```

```
export VERSION=
```

## Task1 Check Jenkins pipeline has been configured
```
gcloud config set compute/zone us-east1-b
```
```
git clone https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/sample-app
gcloud container clusters get-credentials jenkins-cd
```
```
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
helm repo add stable https://kubernetes-charts.storage.googleapis.com/  → this might not work… use the next line of code instead…
helm repo add stable https://charts.helm.sh/stable
helm repo update
helm install cd stable/jenkins
kubectl get pods
```
```
export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/component=jenkins-master" -l "app.kubernetes.io/instance=cd" -o jsonpath="{.items[0].metadata.name}")
kubectl port-forward $POD_NAME 8080:8080 >> /dev/null &
printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
```

- configure Credential 
- Create Multibranch pipeline of sample-app 
## Task2 Check that Jenkins has deployed a development pipeline
```
cd sample-app
kubectl create ns production
kubectl apply -f k8s/production -n production
kubectl apply -f k8s/canary -n production
kubectl apply -f k8s/services -n production
```
```
sed -i "s/blue/$COLOUR/g" html.go
```
```
sed -i "s/1.0.0/$VERSION/g" main.go
```
```
kubectl get svc
kubectl get service gceme-frontend -n production
```
Branch Sources: Git
- Project Repository: https://source.developers.google.com/p/[PROJECT_ID]/r/sample-app
- Credentials: qwiklabs service account

## Task3 Check that Jenkins has deployed a canary pipeline
```
git init
git config credential.helper gcloud.sh
git remote add origin https://source.developers.google.com/p/$DEVSHELL_PROJECT_ID/r/sample-app
```
```
git config --global user.email "<user email>"
git config --global user.name "<user name>"
```
```
git add .
git commit -m "initial commit"
git push origin master
```
## Task4 Check that Jenkins has merged a canary pipeline with production
```
git checkout -b new-feature
git add Jenkinsfile html.go main.go
git commit -m "Version 2.0.0"
git push origin new-feature
curl http://localhost:8001/api/v1/namespaces/new-feature/services/gceme-frontend:80/proxy/version
kubectl get service gceme-frontend -n production
git checkout -b canary
git push origin canary
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
git checkout master
git push origin master
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
while true; do curl http://$FRONTEND_SERVICE_IP/version; sleep 1; done
kubectl get service gceme-frontend -n production
git merge canary
git push origin master
export FRONTEND_SERVICE_IP=$(kubectl get -o \
jsonpath="{.status.loadBalancer.ingress[0].ip}" --namespace=production services gceme-frontend)
```

# Congratulations you've completed your challenge lab
