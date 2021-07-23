## OPENSHIFT DYNAMIC STORAGE
2.03  
```
oc adm top nodes  
oc adm node-logs -u crio node-name
oc debug node/node-name
skopeo inspect
oc get events
oc whoami -t

```

2.05 In this lab we will attach a PV to a deployment  
```
lab install-storage start
source /usr/local/etc/ocp4.config
oc login -u kubeadmin -p ${RHT_OCP4_KUBEADM_PASSWD}
Create a project called install-storage
Create an app with 
name postgresql-persistent \
registry.redhat.io/rhel8/postgresql-12:1-43 \ no anda este
registry.access.redhat.com/rhscl/postgresql-95-rhel7 \
-e POSTGRESQL_USER=redhat \
-e POSTGRESQL_PASSWORD=redhat123 \
-e POSTGRESQL_DATABASE=persistentdb

Add a persistent storage with
name postgresql-storage
class nfs-storage
rwo
size 10Gi
mount path /var/lib/psql
claim name postgresql-storage

Populate data to pvc with ~/DO280/labs/install-storage/init_data.sh
Check ~/DO280/labs/install-storage/check_data.sh 
Remove deploy and all

Create another app with the same vars called postgresql-persistent2
Attach pv again
Delete all deploy and pvc
cd
lab install-storage finish
```

### SOLUTION
```
oc new-app --name postgresql-persistent \
   --docker-image registry.redhat.io/rhel8/postgresql-12:1-43 \
   -e POSTGRESQL_USER=redhat \
   -e POSTGRESQL_PASSWORD=redhat123 \
   -e POSTGRESQL_DATABASE=persistentdb
oc set volume dc/ --add --name postgresql-storage -t pvc --claim-size 10G --claim-class nfs-storage --claim-mode rwo --claim-name postgresql-storage -m /var/lib/psql 
oc get pvc
oc get pv
oc delete all -l app=postgresql-persistent


oc new-app --name postgresql-persistent2 \
   --docker-image registry.redhat.io/rhel8/postgresql-12:1-43 \
   -e POSTGRESQL_USER=redhat \
   -e POSTGRESQL_PASSWORD=redhat123 \
   -e POSTGRESQL_DATABASE=persistentdb

oc set volumes  deployment/postgresql-persistent2 --add --name postgresql-storage -t pvc  --claim-name postgresql-storage --m /var/lib/pgsql
oc delete all -l app=postgresql-persistent
oc delete pvc postgresql-storage
lab install-storage finish

```

# VOLUMES
```
deployment
spec:
  containers:
    volumeMounts:
    - mountPath: /var/lib/psql
      name: mosco
  volumes:
  - name: mosco
    persistentVolumeClaim:
     claimName: mosco-pvc
```
