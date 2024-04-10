## LAB: K8s Creating Pod - Declarative Way (With Yaml File)

This scenario shows:
- how to create basic K8s pod using yaml file,
- how to get more information about pod (to solve troubleshooting),


### Steps

- Run minikube  (in this scenario, K8s runs on WSL2- Ubuntu 20.04) ("minikube start")

  ![image](https://user-images.githubusercontent.com/10358317/153183333-371fe598-d5a4-4b86-9b5d-9e33f35063cc.png)
  
- Create Yaml file (pod1.yaml) in your directory and copy the below definition into the file:

```
apiVersion: v1      
kind: Pod                         # type of K8s object: Pod
metadata:
  name: firstpod                  # name of pod
  labels:
    app: frontend                 # label pod with "app:frontend"   
spec:
  containers: 
  - name: nginx                   
    image: nginx:latest           # image name:image version, nginx downloads from DockerHub
    ports:
    - containerPort: 80           # open ports in the container
    env:                          # environment variables
      - name: USER
        value: "username"
```
![image](https://user-images.githubusercontent.com/10358317/153674646-8997eb99-12b9-4394-91f2-2de4032ee3db.png)


 - Apply/run the file to create pod in declerative way ("kubectl apply -f pod1.yaml"):
   
   ![image](https://user-images.githubusercontent.com/10358317/153198471-55d92940-1141-4e04-a701-6356daaf0181.png)
  
- Describe firstpod ("kubectl describe pods firstpod"):

  ![image](https://user-images.githubusercontent.com/10358317/153199893-95bfbef0-61b4-4c41-bd89-481d976c272c.png)

- Delete pod and get all pods in the default namepace  ("kubectl delete -f pod1.yaml"):

  ![image](https://user-images.githubusercontent.com/10358317/153200081-3f7823a8-e5d0-4143-aac4-157948fe2a61.png)
  
 - If you want to delete minikube  ("minikube delete"):
   
   ![image](https://user-images.githubusercontent.com/10358317/153200584-01971754-0739-4c8f-8446-d2d3ab5bed31.png)

## LAB: K8s Configmap

This scenario shows:
- how to create config map (declerative way),
- how to use configmap: volume and environment variable,
- how to create configmap with command (imperative way),
- how to get/delete configmap


### Steps

- Run minikube  (in this scenario, K8s runs on WSL2- Ubuntu 20.04) ("minikube start")

![image](https://user-images.githubusercontent.com/10358317/153183333-371fe598-d5a4-4b86-9b5d-9e33f35063cc.png)

- Create Yaml file (config.yaml) in your directory and copy the below definition into the file:

``` 
apiVersion: v1
kind: ConfigMap
metadata:
  name: myconfigmap               
data:
  db_server: "db.example.com"        # configmap key-value parameters
  database: "mydatabase"
  site.settings: |
    color=blue
    padding:25px
---
apiVersion: v1
kind: Pod
metadata:
  name: configmappod
spec:
  containers:
  - name: configmapcontainer
    image: nginx
    env:                             # configmap using environment variable
      - name: DB_SERVER
        valueFrom:
          configMapKeyRef:           
            name: myconfigmap        # configmap name, from "myconfigmap" 
            key: db_server
      - name: DATABASE
        valueFrom:
          configMapKeyRef:
            name: myconfigmap
            key: database
    volumeMounts:
      - name: config-vol
        mountPath: "/config"
        readOnly: true
  volumes:
    - name: config-vol               # transfer configmap parameters using volume
      configMap:
        name: myconfigmap
```

![image](https://user-images.githubusercontent.com/10358317/154719668-bcd3bdb2-c102-489e-8049-747fd97126f3.png)

- Create configmap and the pod:

![image](https://user-images.githubusercontent.com/10358317/153645965-84f8fe93-e73e-4468-bce4-4d2c3f49546f.png)

- Run bash in the pod:

![image](https://user-images.githubusercontent.com/10358317/153647020-54a0cf44-582f-4aab-8375-18c9d82ca494.png)

- Define configmap with imperative way (--from-file and --from-literal) (create a file and put into "theme=dark")

```
kubectl create configmap myconfigmap2 --from-literal=background=red --from-file=theme.txt
```

![image](https://user-images.githubusercontent.com/10358317/153647730-cf7e1545-ffbf-4fe8-b87e-6f1b59ef71df.png)

- Delete configmap:

![image](https://user-images.githubusercontent.com/10358317/153647842-87c9b154-fb1e-40a4-893a-700a80c94161.png)
## LAB: K8s Daemon Sets

This scenario shows how K8s Daemonsets work on minikube by adding new nodes

### Steps

- Copy and save (below) as file on your PC (daemonset.yaml). 

```     
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: logdaemonset
  labels:
    app: fluentd-logging
spec:
  selector:
    matchLabels:                                                 # label selector should be same labels in the template (template > metadata > labels)
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master                      # this toleration is to have the daemonset runnable on master nodes
        effect: NoSchedule                                       # remove it if your masters can't run pods  
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2      # installing fluentd elasticsearch on each nodes
        resources:
          limits:
            memory: 200Mi                                        # resource limitations configured           
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:                                            # definition of volumeMounts for each pod 
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:                                                   # ephemerial volumes on node (hostpath defined)   
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers    
```

![image](https://user-images.githubusercontent.com/10358317/154733287-2c65a70a-2d9f-4b69-969e-8e2938ce425d.png)

- Create daemonset on minikube:

![image](https://user-images.githubusercontent.com/10358317/152146006-265e0595-cdf5-43c7-aea2-5437700323fd.png)

- Run watch command on Linux: "watch kubectl get daemonset", on Win: "kubectl get daemonset -w"

![image](https://user-images.githubusercontent.com/10358317/152146266-00d1f1b8-f2dc-495f-ab35-15e3d1629278.png)

- Add new node on the cluster:

![image](https://user-images.githubusercontent.com/10358317/152146458-14a66e8a-fcad-4a15-ac3e-6df1af4a43a4.png)

- To see, app runs automatically on the new node:

![image](https://user-images.githubusercontent.com/10358317/152147031-b934d393-8caf-49c3-ac4c-3b704f2d646a.png)

- Add new node (3rd):

![image](https://user-images.githubusercontent.com/10358317/152151984-ac8fd54c-676d-4be4-b2f1-4356613a8fed.png)

- Now daemonset have 3rd node:

![image](https://user-images.githubusercontent.com/10358317/152152156-c8cd559e-48dc-4ea3-85c9-6da7fbeb0794.png)

- Delete one of the pod:

![image](https://user-images.githubusercontent.com/10358317/152152437-7c883cd5-e809-4386-8832-362a612acf5f.png)

- Pod deletion can be seen here:

![image](https://user-images.githubusercontent.com/10358317/152152613-854c5340-c73b-4d72-bd08-951aa640d8ad.png)

- Daemonset create new pod automatically:

![image](https://user-images.githubusercontent.com/10358317/152152744-9f14751b-e214-4621-8208-1cb5437b6d71.png)

- See the nodes resource on dashboard:

![image](https://user-images.githubusercontent.com/10358317/152153072-5e53cd9c-42ba-4f50-85d8-c82ea1e39752.png)

- Delete nodes and delete daemonset:

![image](https://user-images.githubusercontent.com/10358317/152153355-b98bca05-87cd-46d2-a26d-eb614ca263ca.png)



