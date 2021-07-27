# NetworkPolicy resources should include:
https://www.opentlc.com/labs/ocp4_advanced_deployment/07_1_Network_Policy_Solution_Lab.html  
https://docs.openshift.com/container-platform/4.1/networking/configuring-networkpolicy.html

1. deny-by-default
2. allow-same-namespace
3. allow-cakephp from frontend to db
        Allow for MySQL database pods
        Allow ingress to ports 3306/tcp from cakephp-example-frontend namespace
4. Allow Https, http to pod with label role=frontend
5. allo traffic from namespace and pod name to podname

```
1.
spec: 
   podSelector: {}

2.
spec:
  podSelector: {}
  ingress:
  - from:
     - podSelector: {}

3.
spec:
  podSelector:
    matchLabels: 
     name: mysql
  ingress:
  - from:
     - namespaceSelector: 
        matchLabels: 
         name: frontend
    ports:
     - port: 3306
       protocol: TCP

4.
spec:
  podSelector:
    matchLabels:
      role: frontend
  ingress:
  - ports:
    - protocol: TCP
      port: 80
    - protocol: TCP
      port: 443
5.
spec:
  podSelector:
    matchLabels:
      name: test-pods
  ingress:
    - from:
      - namespaceSelector:
          matchLabels:
            project: project_name
        podSelector:
          matchLabels:
            name: test-pods

```
