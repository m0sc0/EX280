## SDN
### 5.02

```
lab network-sdn start
Login with developer, and create project called network-sdn
cd ~/DO280/labs/network-sdn
Create todo-db.yaml and cp db-data.sql to mysql pod at tmp/
Copy yo items db de sql
Create todo-frontend.yaml
Expose frontend svc  --hostname todo.apps.ocp4.example.com
Oc Debug why is not working (curl -v telnet://172.30.103.29:3306 and :8080/todo/ to frontend)
Use if no curl registry.access.redhat.com/ubi8/ubi:8.0
Note: On fixed from worstation I can curl url, but not the service, service is only from inside sdn
```


### SOLUTION
```
lab network-sdn start
oc login -u developer -p developer
oc new-project network-sdn
cd ~/DO280/labs/network-sdn
oc create -f todo-db.yaml
oc cp db-data.sql mysql-94dc6645b-hjjqb:/tmp/
mysql -u root items < /tmp/db-data.sql
oc create -f todo-frontend.yaml
oc expose service frontend   --hostname todo.apps.ocp4.example.com
oc logs frontend
curl -v telnet://172.30.103.29:3306
oc debug -t deployment/mysql  --image registry.access.redhat.com/ubi8/ubi:8.0 (because have not curl)

oc describe svc frontend ( It shows Enpoints: <none>)
oc edit svc/frontend change name: api to name: frontend
lab network-sdn finish
```
### 5.04  
https://www.openshift.com/blog/self-serviced-end-to-end-encryption-approaches-for-applications-deployed-in-openshift  
The --port in oc route passthrough is the port of the container to route.

```
lab network-ingress start
Login as developer   
Create project called network-ingress
Create from file ~/DO280/labs/network-ingress/todo-app-v1.yaml
Expose svc todo-http with hostname todo-http.apps.ocp4.example.com
Test url
Intercept comunication
Now create a secure route with edge called todo-https  hostname todo-https.apps.ocp4.example.com
curl
Test with firefox
cd ~/DO280/labs/network-ingress
Login as admin
Extract router secret and use it with curl crt (help get secrets all namespaces router)

Login again with developer and see that the comunication between pod and router is not encripted use debug with registry.access.redhat.com/ubi8/ubi:8.0
Delete edge route
cd certs
Generate private key called training.key 2048
CSR with name training.csr with key training.key subjet "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/CN=todo-https.apps.ocp4.example.com"
Generate signed certificate with 1825 days sha256 
Create secret tls called todo-certs  with crt and key 
Create todo-app-v2.yaml (add volume and mount to /usr/local/etc/ssl/certs)
Create passtrought route port 8443 hostname todo-https.apps.ocp4.example.com
curl with certs/training-CA.pem  the route
cd
oc delete project network-ingress
lab network-ingress finish



```


### SOLUTION
```
lab network-ingress start
oc login -u developer -p developer
oc new-project network-ingress
oc create -f  ~/DO280/labs/network-ingress/todo-app-v1.yaml
oc expose svc todo-http --hostname todo-http.apps.ocp4.example.com
sudo tcpdump -i eth0 -A  -n port 80 | grep js
cd ~/DO280/labs/network-ingress
oc create route edge todo-https  --service todo-http  --hostname todo-https.apps.ocp4.example.com
curl  https://todo-https.apps.ocp4.example.com
oc login -u admin -p redhat
oc extract secrets/router-ca --keys tls.crt -n openshift-ingress-operator
curl -I -v   --cacert tls.crt https://todo-https.apps.ocp4.example.com
oc login -u developer -p developer
oc get svc todo-http   -o jsonpath="{.spec.clusterIP}{'\n'}"
oc debug -t deployment/todo-http  --image registry.access.redhat.com/ubi8/ubi:8.0
curl -v 172.30.102.29 (is http behind the router to the pod)
oc delete route todo-https
cd certs; ls
openssl genrsa -out training.key 2048
openssl req -new  -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/CN=todo-https.apps.ocp4.example.com"   -key training.key -out training.csr
openssl x509 -req -in training.csr   -passin file:passphrase.txt    -CA training-CA.pem -CAkey training-CA.key -CAcreateserial    -out training.crt -days 1825 -sha256 -extfile training.ext
cd ~/DO280/labs/network-ingress
oc create secret tls todo-certs    --cert certs/training.crt   --key certs/training.key
oc create -f todo-app-v2.yaml
(If ask add secret to deploy do: oc set volume deployment/todo-https --add -m /usr/local/etc/ssl/certs --secret-name todo-certs --name tls-certs)
oc describe pod todo-https-xxxx
oc create route passthrough todo-https  --service todo-https --port 8443   --hostname todo-https.apps.ocp4.example.com
curl -vvI   --cacert certs/training-CA.pem   https://todo-https.apps.ocp4.example.com
cd
oc delete project network-ingress
lab network-ingress finish

```

### 5.06 NETWORK POLICIES DENY TRAFFIC
```
lab network-policy start
Login as dev
Create network-policy project
Create app called hello and test from quay.io/redhattraining/hello-world-nginx:v1.0
Expose service hello
Open a second therm and run ~/DO280/labs/network-policy/display-project-info.sh
curl -s hello-network-policy.apps.ocp4.example.com | grep Hello
Go to pods a curl to service ip and pod ip


Create new project network-test
Create app called sample-app quay.io/redhattraining/hello-world-nginx:v1.0
Run a second term with ~/DO280/labs/network-policy/display-project-info.sh
In sample pod test that it see pod hello in diferents namespaces curl helloip:8080 |  grep Hello
Also to pod test

Switch to network-policy project and edit /DO280/labs/network-policy/deny-all.yaml
Create file
Verify that from worker cant curl hello-network-policy.apps.ocp4.example.com
Allow ~/DO280/labs/network-policy/allow-specific.yaml 
Allow traffic from  sample-app in network-test to hello pod , port 8080 TCP
(Help: it says traffic to hello pod, so network policy must be in hello namespace)

Allow traffic to expose route from host
Login with admin and label name=network-test th network-test namespace to match the allow-specific
Test curl
cd
lab network-policy finish

```

### SOLUTION
```
lab network-policy start
oc login -u developer -p developer
oc new-project network-policy
oc new-app --name hello --docker-image quay.io/redhattraining/hello-world-nginx:v1.0
oc new-app --name test  --docker-image  quay.io/redhattraining/hello-world-nginx:v1.0
oc expose service hello
Run in second term ~/DO280/labs/network-policy/display-project-info.sh
curl -s hello-network-policy.apps.ocp4.example.com | grep Hello

oc new-project network-test
oc new-app --name sample-app --docker-image   quay.io/redhattraining/hello-world-nginx:v1.0
oc rsh sample-app-d5f945-spx9q curl 10.helloip:8080 |  grep Hello
oc rsh sample-app-d5f945-spx9q curl 10.testip:8080 |  grep Hello


oc project network-policy
cd ~/DO280/labs/network-policy/
vi deny-all.yaml (spec:podSelector: {})
oc create -f deny-all.yaml
curl -s hello-network-policy.apps.ocp4.example.com | grep Hello
oc create -n network-policy -f   allow-specific.yaml

oc login -u admin -p redhat
oc label namespace network-test  name=network-test

cd 
lab network-policy finish

```






### 5.07 TLS PASSTHROUGHT 
```
1)Create lab without restrictions
lab network-review start

Login as developer
New project network-review
cd ~/DO280/labs/network-review/
add 'quay.io/redhattraining/php-ssl:v1.0' to php-http.yaml containerPort: 8080
Create resourse
Create route called php-http.apps.ocp4.example.com
Test with firefox, it is unsecure!!

2)
vi ~/DO280/labs/network-review/deny-all.yaml with an empty pod selector to target all pods in the namespace
Create resourse deny-all.yaml
Test there is no access from workstation
Create a network policy to allow ingress traffic to routes in the network-review namespace edit ~/DO280/labs/network-review/allow-from-openshift-ingress.yaml
ingress with namespaceSelector and this match label network.openshift.io/policy-group: ingress  (use oc explain)
oc login -u admin -p redhat
Label namespace default  network.openshift.io/policy-group=ingress
curl -s http://php-http.apps.ocp4.example.com | grep "PHP"

3) Secure with ca
Login as dev
cd certs
Gen new certificate request with  training.key with subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/CN=php-https.apps.ocp4.example.com" called  training.csr
Gen the x509 certificate with file:passphrase days 3650 

cd ~/DO280/labs/network-review
create secret tls called php-certs  certs/training.crt  certs/training.key
Get secrets
vi ~/DO280/labs/network-review/php-https.yaml add image: 'quay.io/redhattraining/php-ssl:v1.1' containerPort: 8443  secretName: php-certs
create php-https.yaml
Create route passthrough called php-https  service php-https port 8443 hostname php-https.apps.ocp4.example.com
oc get routes
firefox https://php-https.apps.ocp4.example.com and curl -v --cacert certs/training-CA.pem https://php-https.apps.ocp4.example.com
cd
lab network-review grade
lab network-review finish


```



### SOLUTION

```
1)Create lab without restrictions
lab network-review start
oc login -u developer -p developer
oc new-project network-review
cd ~/DO280/labs/network-review/
vi php-http.yaml and add image: 'quay.io/redhattraining/php-ssl:v1.0', containerPort: 8080
oc create -f php-http.yaml
oc expose svc php-http --hostname php-http.apps.ocp4.example.com
Test with firefox

2)Deny all and only from HostNetwork endpoint strategy, label the default namespace with the network.openshift.io/policy-group=ingress label.
vi ~/DO280/labs/network-review/deny-all.yaml with an empty pod selector to target all pods in the namespace
spec:
  podSelector: {}

oc create -f deny-all.yaml
curl http://php-http.apps.ocp4.example.com
vi ~/DO280/labs/network-review/allow-from-openshift-ingress.yaml
  ingress:
  - from:
     - namespaceSelector:
          matchLabels:
             network.openshift.io/policy-group: ingress
oc create -f allow-from-openshift-ingress.yaml
oc login -u admin -p redhat
oc label namespace default  network.openshift.io/policy-group=ingress
curl -s http://php-http.apps.ocp4.example.com | grep "PHP"

3) Secure with ca
oc login -u developer -p developer
cd certs
openssl req -new -key training.key \
    -subj "/C=US/ST=North Carolina/L=Raleigh/O=Red Hat/\
    CN=php-https.apps.ocp4.example.com" \
    -out training.csr

openssl x509 -req -in training.csr \
    -CA training-CA.pem -CAkey training-CA.key -CAcreateserial \
    -passin file:passphrase.txt \
    -out training.crt -days 3650 -sha256 -extfile training.ext

cd ~/DO280/labs/network-review
oc create secret tls php-certs --cert certs/training.crt  --key certs/training.key
oc get secrets
vi ~/DO280/labs/network-review/php-https.yaml add image: 'quay.io/redhattraining/php-ssl:v1.1' containerPort: 8443  secretName: php-certs
oc create -f php-https.yaml
oc create route passthrough php-https  --service php-https --port 8443 --hostname php-https.apps.ocp4.example.com
oc get routes
firefox https://php-https.apps.ocp4.example.com and curl -v --cacert certs/training-CA.pem https://php-https.apps.ocp4.example.com
cd
lab network-review grade
lab network-review finish


```

