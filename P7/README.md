# Roles

### Role-based access control (RBAC)  
```
admin:            A project manager. If used in a local binding, an admin has rights to view any resource in the project and modify any resource in the project except for quota.
* self-provisioner: A user that can create their own projects.
* cluster-admin:    A super-user that can perform any action in any project. When bound to a user with a local binding, they have full control over quota and every action on every resource in the project.

basic-user: 	  A user that can get basic information about projects and users.
cluster-status:   A user that can get basic cluster status information.
edit:             A user that can modify most objects in a project but does not have the power to view or modify roles or bindings.(can create new-app)
view: 		  A user who cannot make any modifications, but can see most objects in a project. They cannot view or modify roles or bindings.

* only this can create projects
** edit can create new-apps, also admin and self-provisioner
```

oc get clusterrolebindings |grep self
oc describe clusterolebindings self-provisioner
oc get clusterrolebinfings self-provisioner -o yaml > backup.yaml

### Only user unit can create projects
oc adm policy remove-cluster-role-from-group self-provisioner system:authentitcated:oauth
oc adm policy add-cluster-role-to-user self-provisioner unit

### Add group 
oc adm groups new manages,tester,developers

### Add role that can view cluster status to user 
oc adm policy add-cluster-role-to-user cluster-status user
 
### Add role that can modify resoruces in a project to user developer in project test
oc adm polocy add-role-to-group admin developers -n test

### Add role modify objects to user developers in project test
oc adm polocy add-role-to-group edit developers -n test

### Add role that can view objets in project to user developers in project virgin
oc adm polocy add-role-to-group edit developers -n virgin

