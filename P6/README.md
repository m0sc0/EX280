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



### 6.04
```
1)
lab schedule-limit start
As developer create project schedule-limit
Create a deployment  called hello-limit with image quay.io/redhattraining/hello-world-nginx:v1.0 and save it to ~/DO280/labs/schedule-limit/hello-limit.yaml
Edit an add 3 cpu and 20M of memory
Create the deploy and save-config
Get events and solve issue "1200m"
Get pods and scale to 4 replicas hello-limit
Delete all with label app=hello-limit


2)
Create the resource ~/DO280/labs/schedule-limit/loadtest.yaml
curl -X GET   http://loadtest.apps.ocp4.example.com/api/loadtest/v1/mem/150/60
http://loadtest.apps.ocp4.example.com/api/loadtest/v1/mem/200/60

3) 
Login as admin/redhat
Create quota called project-quota with har cpu 3, mem 1G and configmaps 3 in namespace schedule-limit
Login as developer
Generate 4 configs maps

4)
Login as admin/redhat
create-bootstrap-project-template   -o yaml > /tmp/project-template.yaml
Add 3cpu and 10Gi in hard of quota
Add limit rangue 
name: ${PROJECT_NAME}-limit and Quota
defaultRequest: cpu: 30m memory: 30M
oc create -f /tmp/project-template.yaml   -n openshift-config

oc edit projects.config.openshift.io/cluster
spec:
  projectRequestTemplate:
    name: project-request
New-project template-test
Get resourcequotas,limitranges
delete project schedule-limit
delete project template-test
lab schedule-limit finish



```

### SOLUTION
```
1)
lab schedule-limit start
oc login -u developer -p developer 
oc new-project schedule-limit
oc create deployment hello-limit   --image quay.io/redhattraining/hello-world-nginx:v1.0   --dry-run=client -o yaml > ~/DO280/labs/schedule-limit/hello-limit.yaml
vi ~/DO280/labs/schedule-limit/hello-limit.yaml
        name: hello-world-nginx
        resources:
          requests:
            cpu: "3"
            memory: 20Mi
oc create --save-config  -f ~/DO280/labs/schedule-limit/hello-limit.yaml
oc get pods
oc get events --field-selector type=Warning
vim ~/DO280/labs/schedule-limit/hello-limit.yaml
          requests:
            cpu: "1200m"
            memory: 20Mi
oc get pods
oc scale --replicas 4 deployment/hello-limit
oc get pods
oc get events --field-selector type=Warning
oc delete all -l app=hello-limit


2)
oc create --save-config  -f ~/DO280/labs/schedule-limit/loadtest.yaml
oc get routes
curl -X GET   http://loadtest.apps.ocp4.example.com/api/loadtest/v1/mem/150/60
oc adm top pod  
curl -X GET  http://loadtest.apps.ocp4.example.com/api/loadtest/v1/mem/200/60
watch oc get pods 
oc delete all -l app=loadtest


3)
oc login -u admin -p redhat
oc create quota project-quota  --hard cpu="3",memory="1G",configmaps="3"   -n schedule-limit
oc login -u developer -p developer

for X in {1..4}
    do
    oc create configmap my-config${X} --from-literal key${X}=value${X}
   done

4)
oc login -u admin -p redhat
oc adm create-bootstrap-project-template   -o yaml > /tmp/project-template.yaml
vi /home/student/DO280/labs/schedule-limit/quota-limits.yaml

- apiVersion: v1
  kind: ResourceQuota
  metadata:
    name: ${PROJECT_NAME}-quota
  spec:
    hard:
      cpu: 3
      memory: 10G
- apiVersion: v1
  kind: LimitRange
  metadata:
    name: ${PROJECT_NAME}-limits
  spec:
    limits:
      - type: Container
        defaultRequest:
          cpu: 30m
          memory: 30M

oc create -f /tmp/project-template.yaml   -n openshift-config
oc edit projects.config.openshift.io/cluster
spec:
  projectRequestTemplate:
    name: project-request
watch oc get pods -n openshift-apiserver
oc new-project template-test
oc get resourcequotas,limitranges
oc delete project schedule-limit
oc delete project template-test
lab schedule-limit finish

```



### 6.06
```
1)
lab schedule-scale start
Login as dev
New project schedule-scale
Edit ~/DO280/labs/schedule-scale/loadtest.yaml  requests cpu 25m, mem 25Mi limits cpu:100m, mem 100Mi
Create app
Describe limits in pod
Scale to 5 replicas, then to 1
Create a horizontal autoscaler max 10 pods when cpu exceds 50% , and min 2 pods
Watch horizontal autoscaler
Add more load curl -X GET http://loadtest-schedule-scale.apps.ocp4.example.com/api/loadtest/v1/cpu/1



2)
Create new app called scalind quay.io/redhattraining/scaling:v1.0 as deployment config
Expose
scale to 3 replicas
Get route and firefox
 ~/DO280/labs/schedule-scale/curl-route.sh
oc delete project schedule-scale
lab schedule-scale finish



```


### SOLUTION
```
1)
lab schedule-scale start
oc login -u developer -p developer
oc new-project schedule-scale
~/DO280/labs/schedule-scale/loadtest.yaml
        resources:
          requests:
            cpu: "25m"
            memory: 25Mi
          limits:
            cpu: "100m"
            memory: 100Mi
oc create --save-config   -f ~/DO280/labs/schedule-scale/loadtest.yaml
oc get pods
oc describe pod/loadtest-5f9565dbfb-jm9md | grep -A2 -E "Limits|Requests"
oc scale --replicas 5 deployment/loadtest
oc get pods
oc scale --replicas 1 deployment/loadtest
oc autoscale deployment/loadtest  --min 2 --max 10 --cpu-percent 50
watch oc get hpa/loadtest
oc get route/loadtest
curl -X GET http://loadtest-schedule-scale.apps.ocp4.example.com/api/loadtest/v1/cpu/1
watch oc get hpa/loadtest



2)
oc new-app --name scaling --as-deployment-config  --docker-image quay.io/redhattraining/scaling:v1.0
oc expose svc/scaling
oc scale --replicas 3 dc/scaling
oc get pods -o wide -l deploymentconfig=scaling
oc get route/scaling
 ~/DO280/labs/schedule-scale/curl-route.sh
Firefox the route
oc get hpa/loadtest 
oc delete project schedule-scale
lab schedule-scale finish



```


