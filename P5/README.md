## SDN
### 5.02

```
lab network-sdn start
Login with developer, and create project called network-sdn
cd ~/DO280/labs/network-sdn
Create todo-db.yaml and cp db-data.sql to mysql pod at tmp/
Create todo-frontend.yaml
Expose frontend svc  --hostname todo.apps.ocp4.example.com
Oc Debug why is not working (curl -v telnet://172.30.103.29:3306 and 8080 to frontend)
Use if no curl registry.access.redhat.com/ubi8/ubi:8.0
```


### SOLUTION
```
lab network-sdn start
oc login -u developer -p developer
oc new-project network-sdn
cd ~/DO280/labs/network-sdn
oc create -f todo-db.yaml
oc cp db-data.sql mysql-94dc6645b-hjjqb:/tmp/
oc create -f todo-frontend.yaml
oc expose service frontend   --hostname todo.apps.ocp4.example.com
oc logs frontend
curl -v telnet://172.30.103.29:3306
oc debug -t deployment/mysql  --image registry.access.redhat.com/ubi8/ubi:8.0 (because have not curl)

oc edit svc/frontend
lab network-sdn finish
```

