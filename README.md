# Microservice communication with RabbitMQ using compose

# NOTE: This is basic kompose usage example with statefulSet, any more adjustment can be made as per your requirement.This should help beginners understand how to get started with your application and the role of StatefulSets in Kubernetes.

# NOTE:
 ```
A StatefulSet is used in Kubernetes for applications that require stable and unique network identifiers, persistent storage, and ordered, graceful deployment and scaling. In your setup, the MySQL database is a suitable candidate for a StatefulSet due to the following reasons:

Reasons for Using StatefulSet for MySQL:
Persistent Storage:

MySQL requires persistent storage to ensure data durability and consistency. When a MySQL pod is restarted, the data must be preserved. StatefulSets, combined with Persistent Volume Claims (PVCs), ensure that each pod gets its own persistent storage that remains consistent across restarts.
Stable Network Identifiers:

StatefulSets provide stable network identities (DNS names) for each of their pods. This is crucial for databases like MySQL where the identity of each node needs to remain stable for the database cluster to function correctly.
Ordered Deployment and Scaling:

StatefulSets ensure that pods are started, stopped, and scaled in an ordered sequence. This ordered deployment and scaling are important for maintaining the consistency and integrity of the MySQL database.
Unique Pod Identity:

Each pod in a StatefulSet has a unique identity that is comprised of a stable, unique hostname. This identity is important for applications like MySQL that might require each instance to be uniquely identifiable.
```

To convert your Docker Compose setup to Kubernetes StatefulSets using `kompose`, follow these steps:

1. **Install Kompose**: Ensure you have `kompose` installed. You can install it via the following command:
    ```sh
    curl -L https://github.com/kubernetes/kompose/releases/download/v1.25.0/kompose-linux-amd64 -o kompose
    chmod +x kompose
    sudo mv ./kompose /usr/local/bin/kompose
    ```

2. **Convert Docker Compose to Kubernetes**:
    Run the `kompose` command to convert your `docker-compose.yml` to Kubernetes manifests.
    ```sh
    kompose convert -f docker-compose.yml
    ```

    This will generate a set of YAML files for Kubernetes resources.

3. **Modify the Generated Kubernetes Manifests**:
    Open the generated YAML files and modify them as needed. Specifically, convert the relevant deployments to StatefulSets.

4. **Apply the Kubernetes Manifests**:
    Apply the generated and modified Kubernetes manifests using `kubectl`.
    ```sh
    kubectl apply -f <generated-file>.yaml
    ```

Here's an example of what you might need to adjust in the generated files:

### 1. Convert Deployments to StatefulSets

For services that require persistent storage, such as MySQL, you should convert the generated Deployment to a StatefulSet. Here's an example:

**mysql_db-deployment.yaml** (generated by kompose):
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-db
spec:
  selector:
    matchLabels:
      app: mysql-db
  serviceName: mysql
  replicas: 1
  template:
    metadata:
      labels:
        app: mysql-db
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_DATABASE
          value: "ims"
        - name: MYSQL_ROOT_PASSWORD
          value: "pesuims"
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

### 2. Ensure Persistent Volumes and Claims

Ensure the Persistent Volume Claims are correctly defined in the generated YAML. For example:

**mysql-db-claim0-persistentvolumeclaim.yaml**:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-persistent-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### 3. Services

Ensure that services are correctly defined to expose the StatefulSets:

**mysql-db-service.yaml**:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql-db
spec:
  ports:
    - port: 3306
  selector:
    app: mysql-db
  clusterIP: None
```

### 4. Deployment for Stateless Applications

The producer and consumers can remain as Deployments, since they do not require persistent storage. Here is an example for the producer:

**producer-deployment.yaml**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: producer
spec:
  replicas: 1
  selector:
    matchLabels:
      app: producer
  template:
    metadata:
      labels:
        app: producer
    spec:
      containers:
      - name: producer
        image: <your-registry>/producer:latest
        ports:
        - containerPort: 5000
        env:
        - name: RABBITMQ_HOST
          value: rabbitmq
        - name: RABBITMQ_PORT
          value: "5672"
        - name: RABBITMQ_USERNAME
          value: guest
        - name: RABBITMQ_PASSWORD
          value: guest
```

Repeat similar steps for all other microservices (consumers).

5. **Applying the Manifests**:
    Finally, apply all the generated and modified manifests:
    ```sh
    kubectl apply -f .
    ```

This setup ensures that your MySQL database runs as a StatefulSet with persistent storage, while other services run as Deployments. Adjustments might be necessary depending on the specifics of your setup and requirements.

----------------------------------------------------------------------------------------over-view-----------------------------------------------------------------------------------------------

# extra steps used:

1. Organize and Modify Manifests
Since Kompose has generated the manifests, you might need to make some adjustments:

Organize Manifests
Move the generated manifests into separate directories for better organization:
```
mkdir -p ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/statefulset
mkdir -p ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment
mkdir -p ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/service
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/mysql-db-deployment.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/statefulset/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/mysql-db-service.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/service/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/producer-deployment.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/producer-service.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/service/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/rabbitmq-deployment.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/rabbitmq-service.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/service/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/consumer-one-deployment.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/consumer-two-deployment.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/consumer-three-deployment.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/consumer-four-deployment.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/db-persistentvolumeclaim.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/statefulset/
mv ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/mysql-db-claim1-persistentvolumeclaim.yaml ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/statefulset/
```
before organishing converted kompose will look like below image
![image](https://github.com/rajath-optit/Application_python_using-_Kompose_with_Statefulset/assets/128474801/c75e80e0-3607-46f0-9bb5-82345c8606eb)

after organishing converted kompose will look like below image
![image](https://github.com/rajath-optit/Application_python_using-_Kompose_with_Statefulset/assets/128474801/7ba4a823-b9d7-400d-8337-58fb7d8f81e4)


step2:
Modify MySQL StatefulSet and PVC
mysql-db-deployment.yaml should be changed to a StatefulSet. You need to manually adjust it since Kompose generates a Deployment for MySQL. Here’s an example StatefulSet configuration:

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-db
spec:
  serviceName: "mysql"
  replicas: 1
  selector:
    matchLabels:
      app: mysql-db
  template:
    metadata:
      labels:
        app: mysql-db
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:
        - name: MYSQL_DATABASE
          value: "ims"
        - name: MYSQL_ROOT_PASSWORD
          value: "pesuims"
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-persistent-storage
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```
Replace mysql-db-deployment.yaml with the above StatefulSet configuration and remove the mysql-db-claim1-persistentvolumeclaim.yaml file if it's redundant.

Update mysql-db-service.yaml to ensure it matches the StatefulSet:
```
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - port: 3306
  clusterIP: None
  selector:
    app: mysql-db
```
2. Apply the Manifests
After organizing and modifying the manifests, apply them to your Kubernetes cluster:

```
# Apply StatefulSet and PVC
kubectl apply -f ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/statefulset/

# Apply Services
kubectl apply -f ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/service/

# Apply Deployments
kubectl apply -f ~/Application_python_using-_Kompose_with_Statefulset/kubernetes/manifest/deployment/
```

3. Verify the Deployment
Check the status of the resources you deployed:
```
kubectl get pods
kubectl get svc
kubectl get statefulsets
kubectl get pvc
```
