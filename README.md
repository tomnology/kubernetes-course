# Kubernetes course stuff Tom Simon
Installation minikube:
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \ && chmod +x minikube
sudo cp minikube /usr/local/bin && rm minikube

Installation Kops:
wget  https://github.com/kubernetes/kops/releases/download/1.10.1/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/

Installation AWS CLI:
sudo apt-add-repository universe
sudo apt-get update
sudo apt-get install python-pip
sudo pip install awscli
aws configure
ls -ahl ~/.aws/

IAM User + S3 bucket einrichten
Route53 DNS

Überprüfung DNS Server der Domain:
sudo apt install bind9-host
host -t NS k8s.inno-on.com

Installation Kubectl:
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
kubectl version

Testen der Context von KubectL:
kubectl config get-contexts

SSH Keys für Kops erzeugen:
ssh-keygen -f .ssh/id_rsa
cat .ssh/id_rsa.pub

Verschönerung Installation:
sudo mv /usr/local/bin/kops-linux-amd64 /usr/local/bin/kops

Kops Kubernetes Cluster managen:
kops create cluster --name=k8s.inno-on.com --state=s3://kops-state-test01 --zones=eu-west-1a --node-count=2 --node-size=t2.micro --master-size=t2.micro --dns-zone=k8s.inno-on.com
kops edit cluster k8s.inno-on.com --state=s3://kops-state-test01
kops update cluster k8s.inno-on.com --yes --state=s3://kops-state-test01
kops delete cluster k8s.inno-on.com --yes --state=s3://kops-state-test01

Kubernetes ersten Service erstellen:
kubectl get services
kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
kubectl expose deployment hello-minikube --type=NodePort

Dann freischalten via AWS securitygroup

NodeIP Adress:Port des services = im webbrowser


Installation Lab-Files + Docker:
git clone https://github.com/wardviaene/docker-demo.git
ls -la
cd docker-demo/
ll
docker build
sudo apt-get install docker.io
sudo docker build .

Oder optional user ubuntu zugriff auf Gruppe docker geben mit: sudo usermod -G docker ubuntu
Dann Logout aus dem SSH und wieder rein mit vagrant ssh

sudo docker run -p 3000:3000 -t 3a93ab605b32 -> startet den container
sudo docker run -p 3000:3000 -it 3a93ab605b32 (control +c killt den container)

dann neues Terminal aufmachen und curl localhost:3000 (gibt hello world zurück)

docker ps -> zeit alle container
docker kill (container ID)
docker images -> zeigt alle images

Account bei Docker Hub erstellen:

docker login
docker tag imageid your-login/docker-demo
docker push your-login/docker-demo

How to write stateless applications: https://12factor.net/

Dann hub.docker.com einloggen, repository erstellen

docker tag 3a93ab605b32 tomnology/transporeon-workshop
docker push tomnology/transporeon-workshop
docker push tomnology/transporeon-workshop:latest (bestimmt den tag, wenn nicht angegeben dann latest)


kubenetes commands:

kubectl get pod
kubectl describe pod <pod>
kubectl expose pod <pod> --port=444 --name=frontend
kubectl port-forward <prod> 8080
kubectl attach <podname> -i
kubectl exec <pod> -- command
kubectl label pods <pod> mylabel=awesome
kubectl run -i --tty busybox --image=busybox --restart=Never -- sh

git clone https://github.com/wardviaene/kubernetes-course
cat first-app/helloworld.yml

kubectl create -f first-app/helloworld.yml
kubectl describe pod nodehelloworld.example.com

kubectl port-forward nodehelloworld.example.com 8081:3000
-> weiteres Terminal öffnen und curl localhost:8081

kubectl expose pod nodehelloworld.example.com --type=NodePort --name=nodehelloworld-service

Security group checken
sollte dann von außen erreichbar sein

kubectl get services

kubectl attach nodehelloworld.example.com (würde logs anzeigen)

kubectl exec nodehelloworld.example.com -- ls /app (zeigt im Container das Verzeichniss /app)
kubectl exec nodehelloworld.example.com -- touch /app/test.txt (wenn wir den container killen ist die Datei weg)

kubectl describe service nodehelloworld-service (zegt weitere Daten zum Service z.B. Endpoint)
NodePort ist statisch

kubectl run -i --tty busybox --image=busybox --restart=Never -- sh -> neuer pod mit neuer Container

telnet 100.96.1.4 3000 (auf den Endpoint von describe service)

Service with loadbalancer:

bei AWS: external loadbalancer hinzufügen lein Problem

für andere Cloud provider: eigenes haproxy / nginx oder expose ports directly

kubectl create -f first-app/helloworld.yml
kubectl create -f first-app/helloworld-service.yml

bevor das funktioniert muss in IAM der master rolle "Full Administator" gegeben werden

kubectl delete svc helloworld-service

Loadbalancer sollte dann erscheinen, dann kann in Route 53 unter Alias Target der ELB konfiguriert werden. (Create record set)

Abschliessend sollte bei Aufruf der angelegten Subdomain "hello world" erscheinen + zeigen das Security Groups angepasst wurden (Nodes können in SG von LB sprechen)

Skalierung Replication Controller:

kubectl get node
cat kubernetes-course/replication-controller/helloworld-repl-controller.yml
kubectl create -f kubernetes-course/replication-controller/helloworld-repl-controller.yml
kubectl get pods
kubectl desricbe pod <pod>
kubectl delete pod <pod>
kubectl get pods
kubectl scale --replicas=4 -f kubernetes-course/replication-controller/helloworld-repl-controller.yml
kubectl get rc (replication controller)
kubectl scale --replicas=1 rc/helloworld-controller
kubectl get pods
kubectl delete rc/helloworld-controller
-> auch ohne yaml file werden die replica einstellungen gespeichert

Deployments:
cat deployment/helloworld.yml
kubectl create -f deployment/helloworld.yml
kubectl get deployments
kubectl get rs
kubectl get pods oder mit --show-lables
kubectl rollout status deployment/helloworld-deployment

Login zu docker hub (2. version):

kubectl expose deploment helloworld-deployment --type=NodePort
kubectl get service
kubectl describe service helloworld-deployment (Dann NodePort anschauen)
minikube service helloworld-deployment --url
dann curl auf URL

kubectl set image deployment/helloworld-deployment k8s-demo=wardviane/k8s-demo:2
kubectl rollout status deployment/helloworld-deployment
dann wiedr curl auf url

kubectl rollout undo deployment/helloworld-deployment
wieder curl (geht wieder zurück auf Hello World)

kubectl edit deployment/helloworld-deployment
unter replicas :3
"revisionHistoryLimit: 100"

kubectl set image deployment/helloworld-deployment k8s-demo=wardviane/k8s-demo:2
kubectl rollout history deployment/helloworld-deployment


Create service vom yaml file:ku

kubectl create -f first-app/helloworld.yml
kubectl get pod
kubectl describe pod nodehelloworld.example.com
cat first-app/helloworld-nodeport-service.yml
kubectl create -f first-app/helloworld-nodeport-service.yml
kubectl describe svc helloworld-service
kubectl get svc

Node Selector using labels:
cat deployment/helloworld-nodeselector.yml
kubectl create -f deployment/helloworld-nodeselector.yml
kubectl get deployments

kubectl describe pod
kubectl get nodes
kubectl label nodes ip-172-20-39-248.eu-west-1.compute.internal hardware=high-spec


healthchecks:

cat deployment/helloworld-healtcheck.yml
kubectl create -f deployment/helloworld-healthcheck.yml
kubectl get pods
kubectl describe helloworld-deployment-name des pods
Dann auf Liveness schauen
WEnn failed dann wird er den Pod terminieren und einen neuen bereitstellen -> gut für production use
kubectl edit deployment/helloworld-deployment dann livenessProbe anschauen


readiness probe:
vim deployment/helloworld-healthcheck.yml
kubectl create -f deployment/helloworld-healthcheck.yml && watch -n1 kubectl get pods (wird zeigen die Pods direkt "READY" sind)
erst wenn readiness probe erfolgreich dann wird auf ready gezeigt
vim deployment/helloworld-liveness-readiness.yml
kubectl create -f deployment/helloworld-liveness-readiness.yml && watch -n1 kubectl get pods
vim deployment/helloworld-liveness-readiness.yml (inital delay anzeigen)

Pod State:
kubectl get pod kube-apiserver-ip-172-20-51-88.eu-west-1.compute.internal -n kube-system -o yaml (den master node verwenden und kube-apiserver davor)


Demo Pod Lifecycle:
cd /kubernetes-course/pod-Lifecycle
vi lifecycle.yaml  -> Init Container zeigen, Main container (containers), readniness probe, livenessprobe, lifecycle posstart / prestop zeigen

in einem Terminal: watch -n1 kubectl get pod
in einem weiteren Terminal: kubectl create -f lifecycle.yaml
kubectl exec -it lifecycle-STRING siehe oben vom pod -- tail /timing -f

Secrets Credentials using volumes:
cat deployment/helloworld-secrets.yml
kubectl create -f deployment/helloworld-secrets.yaml
cat deployment/helloworld-secrets-volumes.yml
kubectl create -f deploment/helloworld-secrets-volumes.yml
kubectl get Pods
kubectl desribe pod podname
Kubernetes selber benutzt secrets mit volumes (VolumeMounts -> serviceaccount)
/etc/creds ist von uns
kubectl exec helloworld-deployment-PODNAME -i -t -- /bin/bash
cat /etc/creds/username
cat /etc/creds/password
mount (sieht man die beiden volumes mit crendtials tmpfs)

DEmo Secrets mit wordpress:
Beachten Sie das wir bisher noch stateless arbeiten und sobald die Pods weg sind auch unsere angeben Daten weg
cat wordpress/wordpress-secrets.yml
cat wordpress/wordpress-single-deployment-no-volumes.yml
https://hub.docker.com/_/wordpress/ -> How to use this image
kubectl create -f wordpress/wordpress-secrets.yml
kubectl create -f wordpress/wordpress-single-deployment-no-volumes.yml
kubectl describe pod
cat wordpress/wordpress-service.yml
kubectl create -f wordpress/wordpress-service.yml
Security Groups beachten
kubectl delete pod wordpress-deployment...
kubectl get pods


Web UI in Kops:
cd dashboard
ls -la
cat readme,.md
kubectl apply -f https://rawgit.com/kubernetes/dashboard/master/src/deploy/kubernetes-dashboard.yaml
vi sample-user.yaml
