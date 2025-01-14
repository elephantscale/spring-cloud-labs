# **Lab 17: Deploy and Orchestrate Microservices on Kubernetes**

## **Objective**
Learn how to deploy `UserService` and `OrderService` microservices on Kubernetes. Use Kubernetes objects such as Pods, Services, and Deployments to manage and scale microservices.

---

## **Lab Steps**

### **Part 1: Installing Kubernetes and Minikube**

1. **Install Minikube.**
   - Visit [Minikube Installation](https://minikube.sigs.k8s.io/docs/start/) and follow the instructions for your operating system.

2. **Install Kubectl.**
   - Download Kubectl from [Kubernetes Tools](https://kubernetes.io/docs/tasks/tools/).
   - Follow the installation steps and add it to your system's `PATH`.

3. **Verify Minikube installation.**
   - Run:
     ```bash
     minikube version
     ```
   - Confirm Minikube is installed.

4. **Start Minikube.**
   - Start a Minikube cluster:
     ```bash
     minikube start --driver=docker
     ```

5. **Verify Kubernetes setup.**
   - Check the cluster information:
     ```bash
     kubectl cluster-info
     ```
   - Confirm Minikube is running:
     ```bash
     minikube status
     ```

---

### **Part 2: Preparing Microservices for Deployment**

6. **Ensure Docker images for `UserService` and `OrderService` are available.**
   - Use the Docker images built in **Lab 16** or create new ones.
   - Load the images into Minikube if necessary:
     ```bash
     minikube image load user-service:1.0
     minikube image load order-service:1.0
     ```

7. **Write a Kubernetes Deployment for `UserService`.**
   - Create a file `user-service-deployment.yaml`:
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
     ```

8. **Write a Kubernetes Service for `UserService`.**
   - Create a file `user-service-service.yaml`:
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

9. **Write a Kubernetes Deployment for `OrderService`.**
   - Create a file `order-service-deployment.yaml`:
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
     ```

10. **Write a Kubernetes Service for `OrderService`.**
    - Create a file `order-service-service.yaml`:
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

### **Part 3: Deploying to Kubernetes**

11. **Apply the `UserService` deployment and service.**
    - Run:
      ```bash
      kubectl apply -f user-service-deployment.yaml
      kubectl apply -f user-service-service.yaml
      ```

12. **Apply the `OrderService` deployment and service.**
    - Run:
      ```bash
      kubectl apply -f order-service-deployment.yaml
      kubectl apply -f order-service-service.yaml
      ```

13. **Verify the pods are running.**
    - Check all running pods:
      ```bash
      kubectl get pods
      ```

14. **Verify services are created.**
    - Check the services:
      ```bash
      kubectl get services
      ```

15. **Expose `OrderService` via NodePort.**
    - Update the `order-service-service.yaml` to use `NodePort`:
      ```yaml
      type: NodePort
      ports:
      - protocol: TCP
        port: 8082
        targetPort: 8082
        nodePort: 30002
      ```
    - Reapply the service:
      ```bash
      kubectl apply -f order-service-service.yaml
      ```

---

### **Part 4: Testing the Deployment**

16. **Access the `OrderService` using NodePort.**
    - Use the Minikube IP to access the service:
      ```bash
      minikube ip
      ```
    - Open a browser or use Postman to call:
      ```
      http://<minikube-ip>:30002/orders
      ```

17. **Verify service interaction.**
    - Confirm that `OrderService` fetches data from `UserService`.

---

### **Part 5: Scaling and Managing Deployments**

18. **Scale the `OrderService` deployment.**
    - Scale the replicas to 3:
      ```bash
      kubectl scale deployment order-service --replicas=3
      ```

19. **Check the scaled pods.**
    - Verify that 3 pods are running:
      ```bash
      kubectl get pods -l app=order-service
      ```

20. **Monitor resource usage.**
    - Use the following command to check resource usage:
      ```bash
      kubectl top pods
      ```

21. **Delete the deployments and services.**
    - Clean up the cluster:
      ```bash
      kubectl delete deployment user-service order-service
      kubectl delete service user-service order-service
      ```

---

### **Optional Exercises (20 mins)**

1. **Integrate ConfigMaps.**
   - Use ConfigMaps to externalize configuration for `UserService` and `OrderService`.

2. **Enable Auto-scaling.**
   - Configure Horizontal Pod Autoscaler (HPA) for `OrderService`.

3. **Test Failover.**
   - Simulate a pod failure and observe Kubernetes automatically restarting the failed pod.

---
