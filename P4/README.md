## OPENSHIFT DYNAMIC STORAGE
### 4.02 Secrets envs used by app
```
lab authorization-secrets start
Login with developer/developer
Create new project called authorization-secrets
Create a secret called mysql with --from-literal user=myuser --from-literal password=redhat123 --from-literal database=test_secrets --from-literal hostname=mysql
Create new app called mysql registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47 
Add to deployment the secret with MYSQL prefix
View if issues are solved
Add a volume with the secret to the deployment mounted in /run/secrets/mysql
 
Create a app called quotes with this image quay.io/redhattraining/famous-quotes:2.1
Add QUOTES PREFIX and use the same secret/mysql
```


### SOLUTION
```
lab authorization-secrets start
oc login -u developer -p developer  https://api.ocp4.example.com:6443
oc new-project authorization-secrets
oc create secret generic mysql --from-literal user=myuser --from-literal password=redhat123 --from-literal database=test_secrets --from-literal hostname=mysql
oc new-app --name mysql --docker-image registry.access.redhat.com/rhscl/mysql-57-rhel7:5.7-47
oc set env deployment/mysql --from secret/mysql  --prefix MYSQL_
oc set volume deployment/mysql --add --type secret --mount-path /run/secrets/mysql --secret-name mysql

oc new-app --name quotes  --docker-image quay.io/redhattraining/famous-quotes:2.1
oc set env deployment/quotes --from secret/mysql  --prefix QUOTES_
oc expose service quotes --hostname quotes.apps.ocp4.example.com
oc get route quotes
curl -s   http://quotes.apps.ocp4.example.com/env | grep QUOTES_
oc delete project authorization-secrets
lab authorization-secrets finish
```

### 4.04
 
```
lab authorization-scc start
Login as developer/developer, create project called authorization-scc and new app name gitlab  quay.io/redhattraining/gitlab-ce:8.4.3-ce.0
The pod fails solve it
Create sa caller gitlab-sa
add scc anyuid to the sa
Add sa to deployment
expose route port 80 with hostname gitlab.apps.ocp4.example.com 
Curl gitlab.apps.ocp4.example.com/users/sign_in
Delete project
lab authorization-scc finish


```
### SOLUTION
```
lab authorization-scc start
oc login -u developer -p developer 
oc new-project authorization-scc
oc new-app --name gitlab  --docker-image quay.io/redhattraining/gitlab-ce:8.4.3-ce.0
oc login -u admin -p redhat
oc get pod/gitlab-7d67db7875-gcsjl -o yaml   | oc adm policy scc-subject-review -f -
oc create sa gitlab-sa
oc adm policy add-scc-to-user anyuid -z gitlab-sa
oc login -u developer -p developer
oc set serviceaccount deployment/gitlab gitlab-sa
oc expose service/gitlab --port 80  --hostname gitlab.apps.ocp4.example.com
curl -s    http://gitlab.apps.ocp4.example.com/users/sign_in | grep '<title>'
oc delete project authorization-scc
lab authorization-scc finish
```

### 4.05 4.02 + 4.04
