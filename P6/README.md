### 6.02 Label
```
1)
lab schedule-pods start
Login as developer
New project called schedule-pods
Deploy an app called hello with image quay.io/redhattraining/hello-world-nginx:v1.0
Expose svc hello
Scale 4 replicas

2) 
Login with admin/redhat
label node master01 with env=dev
label node master02 env=prod
verify labels

3)
Login as dev
Edit deployment and add label env=dev
Verify that pods ar in master01
Delete project

4)
remove label env to nodes
login as dev
switch to schedule-pods-ts
get pods and describe node and solve issue
lab schedule-pods finish



```

### SOLUTION
```
1)
lab schedule-pods start
oc login -u developer -p developer
oc new-project schedule-pods
oc new-app --name hello --docker-image quay.io/redhattraining/hello-world-nginx:v1.0
oc expose svc/hello
oc get pods
oc scale --replicas 4 deployment/hello
oc get pods -o wide

2)
oc login -u admin -p redhat
oc label node master01 env=dev
oc label node master02 env=prod
oc get nodes -L env

3)
oc login -u developer -p developer
oc edit deployment/hello
      nodeSelector:
        env: dev
      restartPolicy: Always
oc get pods -o wide
oc delete project schedule-pods

4)
oc login -u admin -p redhat
oc label node -l env env-
oc login -u developer -p developer
oc project schedule-pods-ts
oc get pods
oc describe pod hello-ts-5dbff9f44-8h7c7
oc edit deployment/hello-ts
      nodeSelector:
        client: ACME
oc get pods
lab schedule-pods finish


```

