# **Lab 17: Deploy and Orchestrate Microservices on Kubernetes with ConfigMaps**

## **Objective**
Learn how to deploy `UserService` and `OrderService` microservices on Kubernetes, externalize configurations using ConfigMaps, and manage containerized applications in a scalable and efficient way.

---

## **Lab Steps**

### **Part 1: Installing Kubernetes and Minikube**

1. **Install Minikube.**
   - Visit [Minikube Installation](https://minikube.sigs.k8s.io/docs/start/) and follow the instructions for your operating system.

2. **Install Kubectl.**
   - Download Kubectl from [Kubernetes Tools](https://kubernetes.io/docs/tasks/tools/).
   - Add it to your system's `PATH` for easy access.

3. **Verify Minikube Installation.**
   - Run:
     ```bash
     minikube version
     ```
   - Confirm the installed version.

4. **Start Minikube.**
   - Launch a Minikube cluster:
     ```bash
     minikube start --driver=docker
     ```

5. **Verify Kubernetes Setup.**
   - Check the cluster information:
     ```bash
     kubectl cluster-info
     ```
   - Confirm Minikube status:
     ```bash
     minikube status
     ```

---

### **Part 2: Preparing Microservices**

6. **Ensure Docker Images for Microservices.**
   - Build Docker images for `UserService` and `OrderService` as done in **Lab 16**.
   - Load the images into Minikube:
     ```bash
     minikube image load user-service:1.0
     minikube image load order-service:1.0
     ```

7. **Externalize Configurations Using `application.properties`.**
   - **For `UserService`**, use the following `application.properties`:
     ```properties
     server.port=${SERVER_PORT:8081}
     app.name=${APP_NAME:user-service}
     database.url=${DATABASE_URL:jdbc:mysql://localhost:3306/userdb}
     ```
   - **For `OrderService`**, use the following `application.properties`:
     ```properties
     server.port=${SERVER_PORT:8082}
     app.name=${APP_NAME:order-service}
     user.service.url=${USER_SERVICE_URL:http://user-service:8081/users}
     ```

---

### **Part 3: Creating ConfigMaps**

8. **Create a ConfigMap for `UserService`.**
   - Save the following in a file named `user-service-configmap.yaml`:
     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: user-service-config
     data:
       SERVER_PORT: "8081"
       APP_NAME: "user-service"
       DATABASE_URL: "jdbc:mysql://user-database:3306/userdb"
     ```

9. **Create a ConfigMap for `OrderService`.**
   - Save the following in a file named `order-service-configmap.yaml`:
     ```yaml
     apiVersion: v1
     kind: ConfigMap
     metadata:
       name: order-service-config
     data:
       SERVER_PORT: "8082"
       APP_NAME: "order-service"
       USER_SERVICE_URL: "http://user-service:8081/users"
     ```

10. **Apply ConfigMaps in Kubernetes.**
    - Apply the ConfigMaps to the cluster:
      ```bash
      kubectl apply -f user-service-configmap.yaml
      kubectl apply -f order-service-configmap.yaml
      ```

---

### **Part 4: Writing Kubernetes Manifests**

11. **Create Deployment for `UserService`.**
    - Save the following as `user-service-deployment.yaml`:
      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: user-service
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: user-service
        template:
          metadata:
            labels:
              app: user-service
          spec:
            containers:
            - name: user-service
              image: user-service:1.0
              ports:
              - containerPort: 8081
              envFrom:
              - configMapRef:
                  name: user-service-config
      ```

12. **Create Service for `UserService`.**
    - Save the following as `user-service-service.yaml`:
      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: user-service
      spec:
        selector:
          app: user-service
        ports:
        - protocol: TCP
          port: 8081
          targetPort: 8081
        type: ClusterIP
      ```

13. **Create Deployment for `OrderService`.**
    - Save the following as `order-service-deployment.yaml`:
      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: order-service
      spec:
        replicas: 2
        selector:
          matchLabels:
            app: order-service
        template:
          metadata:
            labels:
              app: order-service
          spec:
            containers:
            - name: order-service
              image: order-service:1.0
              ports:
              - containerPort: 8082
              envFrom:
              - configMapRef:
                  name: order-service-config
      ```

14. **Create Service for `OrderService`.**
    - Save the following as `order-service-service.yaml`:
      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: order-service
      spec:
        selector:
          app: order-service
        ports:
        - protocol: TCP
          port: 8082
          targetPort: 8082
        type: ClusterIP
      ```

---

### **Part 5: Deploying to Kubernetes**

15. **Apply the `UserService` Deployment and Service.**
    - Run:
      ```bash
      kubectl apply -f user-service-deployment.yaml
      kubectl apply -f user-service-service.yaml
      ```

16. **Apply the `OrderService` Deployment and Service.**
    - Run:
      ```bash
      kubectl apply -f order-service-deployment.yaml
      kubectl apply -f order-service-service.yaml
      ```

17. **Verify the Pods and Services.**
    - List all Pods:
      ```bash
      kubectl get pods
      ```
    - List all Services:
      ```bash
      kubectl get services
      ```

---

### **Part 6: Testing and Scaling**

18. **Test the Services.**
    - Access `OrderService` via port-forwarding:
      ```bash
      kubectl port-forward service/order-service 8082:8082
      ```
    - Open a browser or Postman to call:
      ```
      http://localhost:8082/orders
      ```

19. **Scale the Deployments.**
    - Scale `OrderService` to 3 replicas:
      ```bash
      kubectl scale deployment order-service --replicas=3
      ```
    - Verify the updated Pods:
      ```bash
      kubectl get pods -l app=order-service
      ```

20. **Clean Up.**
    - Delete the Deployments and Services:
      ```bash
      kubectl delete deployment user-service order-service
      kubectl delete service user-service order-service
      ```

---

### **Optional Exercises**

1. **Enable Horizontal Pod Autoscaling.**
   - Configure Kubernetes HPA for `OrderService`.

2. **Add ConfigMap Updates.**
   - Update ConfigMaps and observe how Pods reload configurations dynamically.

3. **Simulate Failures.**
   - Kill a Pod and observe Kubernetes restarting it automatically.

---
