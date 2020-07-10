# Monitoring using Prometheus and Grafana
### Monitoring is a necessity in today’s world.
### Here i use Prometheus and Grafana for monitoring our system. Prometheus use for store the metrics and  Grafana is represent in vitual 



## So here we perform below task 

### Integrate Prometheus and Grafana and perform in following way:
### 1. Deploy them as pods on top of Kubernetes by creating resources Deployment, ReplicaSet, Pods or Services
### 2. And make their data to be remain persistent
### 3. And both of them should be exposed to outside world.

### so here push my docker images of Prometheus and Grafana on top of my docker hub account

## Create a Service \
### by default prometheus use port 9090 , so we expose it using service ,so public can access this prometheus 
```
--- 
apiVersion: v1
kind: Service
metadata: 
  lables: 
    app: prom
  name: prom-svc
spec: 
  ports: 
    - 
      port: 9090
      
  selector: 
    app: prom
  type: NodePort
---
```

## Create a PersistentVolumeClaim
```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata: 
  lables:
    app: prom
  name: prom-vol
spec: 
  accessModes: 
    - ReadWriteOnce
  resources: 
    requests: 
      storage: 3Gi
 ---
 ```
 ### here i create a PVC it is Dynamically ceate a PV for us so we don't have create pv manually
 ### i am create single file for creating all resoures for k8s deployment by separet by --- otherwise some time faild to deploy 
## ConfigMap

### A ConfigMap is an API object that lets you store configuration for other objects to use. Unlike most Kubernetes objects that have a spec , a ConfigMap has a ### data section to store items (keys) and their values. The name of a ConfigMap must be a valid DNS subdomain name.
### Using the configmap different ways ,that you can use a ConfigMap to configure a container inside a Pod
### config map use according to you need without config map you can create deployment 
```
apiVersion: v1
data: 
  prometheus.yml: |
      global:
        scrape_interval: 15s
        evaluation_interval: 15s
      alerting:
        alertmanagers:
        - static_configs:
          - targets:
      rule_files:
      scrape_configs:
      - job_name: ‘prometheus’
        
        static_configs:
        - targets: [‘127.0.0.1:9090’]
kind: ConfigMap
metadata: 
  labels: 
    app: prom
  name: prom-config
  ```

 ## Create Deployment for  Prometheus 
 ### here i create deployment and i attach the service and pvc with deplyment 
 ### For creating container  i add my docker image .
 ```
 
 apiVersion: apps/v1
kind: Deployment
metadata: 
  labels: 
    app: prom
  name: prom-deploy
spec: 
  selector: 
    matchLabels: 
      app: prom
  template: 
    metadata: 
      labels: 
        app: prom
      name: prom-deploy
    spec: 
      containers: 
        - 
          image: 9057508163/prometheus
          name: prometheus
          ports: 
            - 
              containerPort: 9090
          volumeMounts: 
            - 
              mountPath: /etc/prometheus/prometheus.yml
              name: prom-config
              subPath: prometheus.yml
            - 
              mountPath: /prometheus
              name: prom-vol
      volumes: 
        - 
          name: prom-vol
          persistentVolumeClaim: 
            claimName: prom-vol
        - 
          name: prom-config
          configMap: 
            name: prom-config
          
---
```
```
root@vishnu:/vv/prom# kubectl apply -f prometheus.yml --validate=false
service/prom-svc created
persistentvolumeclaim/prom-vol created
configmap/prom-config created
deployment.apps/prom-deploy created
root@vishnu:/vv/prom# 
```
