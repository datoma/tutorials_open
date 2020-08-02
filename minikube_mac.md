#Kubernetes minikube Tutorial
https://github.com/kubernetes/minikube

## Vorraussetzung
### Virtualisierung
minikube setzt voraus, dass VT-x/AMD-v virtualization im Bios aktiviert ist. Am einfachsten prüft man dies durch:
    $ sysctl -a | grep machdep.cpu.features | grep VMX
Wenn irgendwas zurückkommt sollte minikube laufen - falls nicht, bitte im Bios prüfen und aktivieren.

### Pakete
Damit minukube läuft müssen folgende Pakete installiert sein:
- kubectl
- docker (for Mac)
- minikube
- virtualbox

Am einfachsten man installiert diese mit brew:
    $ brew update && brew install kubectl && brew cask install docker minikube virtualbox

Anschliessend prüfen, ob alles funktioniert (Ausgabe sollte jeweils die Versionsnummer sein):
    $ docker --version
    $ docker-compose --version
    $ docker-machine --version
    $ minikube version
    $ kubectl version --client

## minikube
#### Installieren
    $ minikube start

Dies dauert - je nach Bandbreite und CPU ein wenig. Am Ende sollte man jedoch so etwas wie folgt als Rückgabe bekommen:
    Starting local Kubernetes v1.10.0 cluster...
    Starting VM...
    Downloading Minikube ISO
     153.08 MB / 153.08 MB [============================================] 100.00% 0s
    Getting VM IP address...
    Moving files into cluster...
    Downloading kubeadm v1.10.0
    Downloading kubelet v1.10.0
    Finished Downloading kubelet v1.10.0
    Finished Downloading kubeadm v1.10.0
    Setting up certs...
    Connecting to cluster...
    Setting up kubeconfig...
    Starting cluster components...
    Kubectl is now configured to use the cluster.
    Loading cached images from config file.

#### Installation prüfen
    $ minikube status 
Ausgabe:
    minikube: Running
    cluster: Running
    kubectl: Correctly Configured: pointing to minikube-vm at xxx.xxx.xxx.xxx

#### Dashboard
Das Dahboard kann entweder direkt im Browser aufgerufen werden, oder man verwendet folgenden Befehl:
    $ minikube dashboard

#### kubectl
#####  cluster-info
FÜr allgeminee Infos über den CLuster einfach 
    $ kubectl cluster-info
eingeben. AUsgabe sieht dann wie folgt aus:
    Kubernetes master is running at https://xxx.xxx.xxx.xxx:8443
    KubeDNS is running at https://xxx.xxx.xxx.xxx:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
Falls der Status "not ready" ist, einfach in ein paar Minuten nochmal probieren. Falls der Status dauerhaft "Not ready" ist, lief irgendwas schief

##### get nodes
Die Knoten unseres Systems können mit
    $ kubectl get nodes
ausgegeben werden: 
    NAME       STATUS    ROLES     AGE       VERSION
    minikube   Ready     master    1m        v1.10.0

##### get pods
Um alle Pods in allen Namespaces auszugegeb
    $ kubectl get pods --all-namespaces
eingeben. Ausgabe sieht dann wie folgt aus:
    NAMESPACE     NAME                                    READY     STATUS    RESTARTS   AGE
    kube-system   etcd-minikube                           1/1       Running   0          3m
    kube-system   kube-addon-manager-minikube             1/1       Running   0          3m
    kube-system   kube-apiserver-minikube                 1/1       Running   0          3m
    kube-system   kube-controller-manager-minikube        1/1       Running   0          3m
    kube-system   kube-dns-86f4d74b45-4c4rk               3/3       Running   0          4m
    kube-system   kube-proxy-9d7hg                        1/1       Running   0          4m
    kube-system   kube-scheduler-minikube                 1/1       Running   0          3m
    kube-system   kubernetes-dashboard-5498ccf677-8lc6l   1/1       Running   0          4m
    kube-system   storage-provisioner                     1/1       Running   0          4m
Die Ausgabe kann abweichen, je nach K8S Version

##### use-context
Setzen des Clusters
    $ kubectl config use-context minikube
Ausgabe
    Switched to context "minikube".
Prüfen des Context via
    $ kubectl config current-context

##### config view
um die Konfiguration des Clusters (kubeconfig) auszugeben, kann folgender Befehl eingesetzt werden:
    $ kubectl config view
Ausgabe:
    apiVersion: v1
    clusters:
    - cluster:
        certificate-authority: /Users/xxx/.minikube/ca.crt
        server: https://xxx.xxx.xxx.xxx:8443
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
        client-certificate: /Users/xxx/.minikube/client.crt
        client-key: /Users/xxx/.minikube/client.key
### Deployment der "Hello" App
Um auf die schnelle eine kleine App zu deployen nutzen wir die "Hello" app von google
    § kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
Das dauert wieder ein wenig und als AUsgabe erhalten wir
    deployment.apps/hello-minikube created

##### get deployments
Umd ie Deployments unseres Clusters anzuzeigen, nutzen wir
    § kubectl get deployments
Ausgabe
    NAME             DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    hello-minikube   1         1         1            1           5m

##### get pods
    § kubectl get pods
Ausgabe
    NAME                             READY     STATUS    RESTARTS   AGE
    hello-minikube-c8b6b4fdc-cpt2v   1/1       Running   0          5m

##### expose
Damit wir die Applikation erreichen können. müssen wir den Service nach aussen verfügbar machen:
    § kubectl expose deployment hello-minikube --type=NodePort
Ausgabe:
    service/hello-minikube exposed

##### get services
Um die Services und Ports, die der CLuster anbietet einsehen zu können, geben wir folgendes ein:
    $ kubectl get services
Ausgabe:
    NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
    hello-minikube   NodePort    10.110.5.33   <none>        8080:31744/TCP   6m
    kubernetes       ClusterIP   10.96.0.1     <none>        443/TCP          4m

##### URL eines Services
Damit wir die URL eines Services bekommen, ist folgendes einzugeben:
    $ minikube service hello-minikube --url
Ausgabe:
    http://xxx.xxx.xxx.xxx:31744

##### Service abfragen
Wir können die URL sowohl in den Browser eingeben, als auch per curl abfragen:
    $ curl $(minikube service hello-minikube --url)
Ausgabe:
    CLIENT VALUES:
    client_address=172.17.0.1
    command=GET
    real path=/
    query=nil
    request_version=1.1
    request_uri=http://xxx.xxx.xxx.xxx:8080/
    
    SERVER VALUES:
    server_version=nginx: 1.10.0 - lua: 10001
    
    HEADERS RECEIVED:
    accept=*/*
    host=xxx.xxx.xxx.xxx:31744
    user-agent=curl/7.54.0
    BODY:
    -no body in request-

### Deployment der App löschen
Um die Applikaiton wieder zu entfernen sind folgende Schritte notwendig:
    $ kubectl delete deployment,service hello-minikube
Ausgabe:
    deployment "hello-minikube" deleted
    service "hello-minikube" deleted

Ob die deinstallation geklappt hat, kann man mittels
    $ kubectl get deployments
    No resources found.
und
    $ kubectl get pods
    No resources found.
prüfen.

## Eigene Docker-Registry aufsetzen
### Environment setzen
##### setzen
Damit Minikube auf Docker zugreifen kann, muss das Environment entsprechend gesetzt werden:
    $ eval $(minikube docker-env)
Man bekommt keine Rückgabe, kann jedoch kontrollieren, in dem man
    $ docker ps
eingibt. Es sollten nun einige Kubernetes Container zu sehen sein, die vorher nicht vorhanden waren.
    85f72f072ec0        k8s.gcr.io/k8s-dns-sidecar-amd64           "/sidecar --v=2 --lo…"   About an hour ago   Up About an hour                             k8s_sidecar_kube-dns-86f4d74b45-4c4rk_kube-system_f0e5d955-8410-11e8-b850-0800276efec0_0
    0b8718a0fe99        k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64     "/dnsmasq-nanny -v=2…"   About an hour ago   Up About an hour                             k8s_dnsmasq_kube-dns-86f4d74b45-4c4rk_kube-system_f0e5d955-8410-11e8-b850-0800276efec0_0
    3bcaa934a8a5        k8s.gcr.io/kubernetes-dashboard-amd64      "/dashboard --insecu…"   About an hour ago   Up About an hour                             k8s_kubernetes-dashboard_kubernetes-dashboard-5498ccf677-8lc6l_kube-system_f1ddde58-8410-11e8-b850-0800276efec0_0
    122309457c20        gcr.io/k8s-minikube/storage-provisioner    "/storage-provisioner"   About an hour ago   Up About an hour                             k8s_storage-provisioner_storage-provisioner_kube-system_f1eaee7e-8410-11e8-b850-0800276efec0_0
    047149890bf4        k8s.gcr.io/k8s-dns-kube-dns-amd64          "/kube-dns --domain=…"   About an hour ago   Up About an hour                             k8s_kubedns_kube-dns-86f4d74b45-4c4rk_kube-system_f0e5d955-8410-11e8-b850-0800276efec0_0
    e2f34959a9eb        k8s.gcr.io/kube-proxy-amd64                "/usr/local/bin/kube…"   About an hour ago   Up About an hour                             k8s_kube-proxy_kube-proxy-9d7hg_kube-system_f0e4ca99-8410-11e8-b850-0800276efec0_0
    ...

##### Zurücksetzen
Das Envoironment kann jederzeit mit
    $ eval $(docker-machine env -u)
zurückgesetzt werden. Wenn man nun 
    $ docker ps
eingobt, sollten die Kubernetes Container weg sein

##### dauerhaft setzen
Am einfachsten ist es, wenn man den eval Befehl in die jeweilige .bash_profile or .zshrc aufnimmt.

### Registry installieren
Die Docker-Registry wird mit folgendem Befehl ausgeführt. Es ist wichtig, dass der obige "Eval"-Befehl zuerst ausgeführt wurde
    $ docker run -d -p 5000:5000 --restart=always --name registry registry:2
Ausgabe:
    Unable to find image 'registry:2' locally
    2: Pulling from library/registry
    4064ffdc82fe: Pull complete 
    c12c92d1c5a2: Pull complete 
    4fbc9b6835cc: Pull complete 
    765973b0f65f: Pull complete 
    3968771a7c3a: Pull complete 
    Digest: sha256:51bb55f23ef7e25ac9b8313b139a8dd45baa832943c8ad8f7da2ddad6355b3c8
    Status: Downloaded newer image for registry:2
    e6e269c214a67baf729ab94932f1660bb0e4ecb90ff427e04c150940293b8b54
Damit läuft nun eine lokale Docker-Registry

## eigene Docker-Apps bauen
### Webserver und HTML-File
##### Files erstellen
    $ mkdir sample-app && cd sample-app && touch Dockerfile index.html sample-app.yml
Dockerfile erstellen
    $ vi Dockerfile

    FROM httpd:2.4-alpine

    COPY ./index.html /usr/local/apache2/htdocs/
index.html erstellen:
    $ vi index.html

    Hello sample k8s world!
sample-app.yml ertsellen:
    $ vi sample-app.yml

    # APP DEPLOYMENT
    
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      labels:
        run: sample-app
      name: sample-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          run: sample-app-exposed
      template:
        metadata:
          labels:
            run: sample-app-exposed
        spec:
          containers:
          - image: localhost:5000/sample-app:0.1.0
            name: sample-app
            ports:
            - containerPort: 80
              protocol: TCP
    
    ---
    
    # APP SERVICE
    
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: sample-app
      name: sample-app
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        run: sample-app-exposed
      type: NodePort

##### bauen
Das Docker-Image bauee
    $ docker build . -t sample-app
Anschliessend das Image in die locale Docker-Registry kopieren
    $ docker tag sample-app localhost:5000/sample-app:0.1.0
Prüfen mit
    $ docker images
Hier sollte das Image nun zwei Mal auftauchen. Einmal in der Rwgistry (localhost:5000) und einmal lokal.

##### Deployen 
Um die App in Kubernetes zu deployen ist foglendes auszuführen:
    $ kubectl create -f sample-app.yml
Ausgabe: 
    deployment.extensions/sample-app created
    service/sample-app created

um das Deployment zu prüfen kann man entweder die KOmmandos oben ausführen (get deploymentsklsr, get pod, ...) oder (bei einem kleineren Cluster)
    $ kubectl get all
Ausgabe:
    NAME                         READY     STATUS    RESTARTS   AGE
    pod/sample-app-9bdf76c69-r92d9   1/1       Running   0          6s
    
    NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
    service/kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP        3m
    service/sample-app       NodePort    10.98.101.3   <none>        80:32741/TCP   6s
    
    NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/sample-app   1         1         1            1           6s
    
    NAME                               DESIRED   CURRENT   READY     AGE
    replicaset.apps/sample-app-9bdf76c69   1         1         1         6s

Die URL des Service kann man wieder mit 
    $ minikube service sample-app --url
erfragen. Fertig.
App entfernen mit
    $ kubectl delete deployment,service sample-app

### NodeJS App
##### Files erstellen
    $ mkdir sample-nodejs && cd sample-nodejs && touch Dockerfile server.js sample-nodejs.yaml
Dockerfile erstellen:
    $ vi Dockerfile
    
    FROM node:6.14.3
    EXPOSE 8080
    COPY server.js .
    CMD node server.js
server.js anlegen:
    $ vi server.js

    var http = require('http');
    var handleRequest = function(request, response){
        console.log("rx request for url:" + request.url);
        response.writeHead(200)
        response.end('Hello World!')
    };

    var www = http.createServer(handleRequest);
    www.listen(8080);
    
sample-nodejs.yaml anlegen:
    $ vi sample-nodejs.yaml
    
    # APP DEPLOYMENT
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      labels:
        run: sample-nodejs
      name: sample-nodejs
    spec:
      replicas: 1
      selector:
        matchLabels:
          run: sample-nodejs-exposed
      template:
        metadata:
          labels:
            run: sample-nodejs-exposed
        spec:
          containers:
          - image: localhost:5000/sample-nodejs:1.0.0
            name: sample-nodejs
            ports:
            - containerPort: 8080
              protocol: TCP
    
    ---
    
    # APP SERVICE
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: sample-nodejs
      name: sample-nodejs
    spec:
      ports:
      - port: 8080
        protocol: TCP
        targetPort: 8080
      selector:
        run: sample-nodejs-exposed
      type: NodePort

##### bauen
    $ docker build . -t sample-nodejs
    $ docker tag sample-nodejs localhost:5000/sample-nodejs:1.0.0

##### Deployen 
    $ kubectl create -f sample-nodejs.yaml
wieder prüfen
    $ kubectl get all
und die URL ermitteln:
    $ minikube service sample-nodejs --url
fertig.

App entfernen mit
    $ kubectl delete deployment,service sample-nodejs


## alles auf Anfang
Um alles wieder zu deinstallieren sind folgende Schritte notwendig:
    $ minikube stop
    $ minikube delete
    $ rm -rf ~/.minikube .kube
    $ brew uninstall kubectl
    $ brew cask uninstall docker virtualbox minikube

## Last Update
07/2018

## Quellen:
https://gist.github.com/kevin-smets/b91a34cea662d0c523968472a81788f7
http://johnmclaughlin.info/learn-kubernetes-using-minikube-docker-macos/
