## OPENSHIFT AUTH
### 3.02 In this lab we will attach an Identity Provider  
```
lab auth-provider start
source /usr/local/etc/ocp4.config
Use htpasswd to populate an auth with
admin/redhat with bcrypt and save it to ~/DO280/labs/auth-provider/htpasswd
Add developer/developer user
oc login -u kubeadmin -p ${RHT_OCP4_KUBEADM_PASSWD} https://api.ocp4.example.com:6443
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
lab auth-provider finish
```

### SOLUTION
```
htpasswd -c -b -B   ~/DO280/labs/auth-provider/htpasswd admin redhat
htpasswd -b  ~/DO280/labs/auth-provider/htpasswd developer developer
o
vi usuarios
cat usuarios | while read USER PWD; do echo $USER $PWD; htpasswd -B -b /home/student/DO280/labs/auth-provider/htpasswd $USER $PWD; done
cat usuarios | while read USER PWD; do echo $USER $PWD; oc login -u $USER -p $PWD; done

oc login -u kubeadmin -p ${RHT_OCP4_KUBEADM_PASSWD} https://api.ocp4.example.com:6443
oc create secret generic localusers --from-file htpasswd=/DO280/labs/auth-provider/htpasswd -n opneshift-config
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
lab auth-provider finish

```

### 3.04 Role Based Access Control  

```
Remove cluster role self-provisioners from all user who are no cluster admins
Add admin role to user leader/redhat in aut-rbac project
Create dev-group with developer user and qa-group with qa-engineer
Add policy edit to dev-group, and view to qa-group.
Login with developer and create a app with httpd:2.44 image then try to scale deployment with 3 replicas

Restore project creation privileges to all users
```

### SOLUTION

```
lab auth-rbac start
oc login -u admin -p redhat
oc get clusterrolebinding
oc describe clusterrolebindings self-provisioners
oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
oc policy add-role-to-user admin leader
oc describe rolebindings | grep -B 9 leader
oc adm groups new dev-group
oc adm groups add-users dev-group developer
oc adm groups new qa-group
oc adm groups add-users qa-group qa-engineer
oc get groups
oc login -u leader -p redhat
oc policy add-role-to-group edit dev-group
oc policy add-role-to-group view qa-group
oc get rolebindings -o wide
oc login -u developer -p developer
oc new-app --name httpd httpd:2.44
oc login -u qa-engineer -p redhat
oc scale deployment httpd --replicas 3

oc adm policy add-cluster-role-to-group  --rolebinding-name self-provisioners  self-provisioner system:authenticated:oauth
lab auth-rbac finish
```

### 3.05 is a mix of 3.02 and 3.04
ToDo 

https://docs.openshift.com/container-platform/4.1/authentication/using-rbac.html
Delete kubeadmin account
oc delete secret kubeadmin -n kube-system 
Add developer to create projects
oc adm policy add-cluster-role-to-user self-provisioner developer
