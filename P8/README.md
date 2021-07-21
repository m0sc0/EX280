# Limits & Quota 


## Limits

https://docs.openshift.com/container-platform/3.6/admin_guide/limits.html  
https://github.com/acsulli/ocp-4-resource-management  

### Steal a limitrange template  
oc get limitranges --all-namespaces  
oc get limitranges mem-limit-range -n default -o yaml > patron.yaml  


```
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "core-resource-limits" 
spec:
  limits:
    - type: "Pod"
      max:
        cpu: "2" 
        memory: "1Gi" 
      min:
        cpu: "200m" 
        memory: "6Mi" 
    - type: "Container"
      max:
        cpu: "2" 
        memory: "1Gi" 
      min:
        cpu: "100m" 
        memory: "4Mi" 
      default:
        cpu: "300m" 
        memory: "200Mi" 
      defaultRequest:
        cpu: "200m" 
        memory: "100Mi" 
      maxLimitRequestRatio:
        cpu: "10" 

+---------------------------------------------------------------------------------------------------------------------+  
| default:  	  The default amount of CPU that a container will be !!!!limited!!!! to use if not specified.  
| defaultRequest: The default amount of CPU that a container will request to use if not specified.  
|  
| default is the limit if pod no specified cpu or memory  
| defaultrequest is the minimun if no especified.  
+---------------------------------------------------------------------------------------------------------------------+  


```
apiVersion: v1
kind: Pod
resources:
  limits:
    cpu: "1"
  requests:
    cpu: 500m
https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-resource-requests-and-limits
The pod with request for 500m and could grow up to the limit 1 cpu
```

## Quota  
https://docs.openshift.com/container-platform/4.7/applications/quotas/quotas-setting-per-project.html  

+---------------------------------------------------------------------------------------------------------------------+  
| cpu == request.cpu												      |  
| memory == request.memory:   The sum of all cpu request of pods 						      |  
| limits.cpu:                 The sum of CPU limits across all pods in a non-terminal state cannot exceed this value. |  
+---------------------------------------------------------------------------------------------------------------------+  

oc create quota quota-test --hard=pods=4,cpu=3,replicationcontrollers=3,services=3,requests.cpu=2,requests.memory=2Gi  


