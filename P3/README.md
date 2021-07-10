## OPENSHIFT AUTH
3.02 In this lab we will attach an Identity Provider
```
lab auth-provider start
source /usr/local/etc/ocp4.config
Use htpasswd to populate an auth with
admin/redhat with bcrypt and save it to ~/DO280/labs/auth-provider/htpasswd
Add developer/developer user
Create a secret from the file called  localusers in namespace openshift-config 
Asign admin user to cluster-admin role

Add to oauth cluster
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: localusers
    mappingMethod: claim
    name: myusers
    type: HTPasswd
Login with admin user test nodes
Login with dev and test nodes
Add manager user with random pass and login
Delete manager user
List users
List identities
Delete all users and identities and the secret
```

### SOLUTION
```
htpasswd -c -b -B   ~/DO280/labs/auth-provider/htpasswd admin redhat
htpasswd -b  ~/DO280/labs/auth-provider/htpasswd developer developer
oc create secret generic localusers --from-file=DO280/labs/auth-provider/htpasswd -n opneshift-config
oc adm policy add-cluster-role-to-user cluster-admin admin
oc edit oauth cluster
oc login -u admin -p redhat
oc get nodes
oc login -u developer -p developer
oc get identity
oc extract secret/localusers -n openshift-config   --to ~/DO280/labs/auth-provider/ --confirm
htpasswd -b ~/DO280/labs/auth-provider/htpasswd manager redhat
oc set data secret/localusers --from-file htpasswd=/home/student/DO280/labs/auth-provider/htpasswd  -n openshift-config
htpasswd -D ~/DO280/labs/auth-provider/htpasswd manager redhat
oc set data secret/localusers --from-file htpasswd=/home/student/DO280/labs/auth-provider/htpasswd  -n openshift-config
oc delete identity "mysusers:manager"
oc delete user manager
oc get users
oc get identity
oc delete user --all
oc delete identity --all
oc delete secret localusers -n openshift-config

```
