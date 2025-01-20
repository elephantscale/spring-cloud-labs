# **Lab 16: Containerize Your Microservices Using Docker and Manage Container Images (Spring Boot 3.4.1)**

## **Objective**
Learn how to containerize two **Spring Boot 3.4.1** microservices (`UserService` and `OrderService`) with **Docker**. You will build Docker images, run containers locally, manage containers (start/stop/remove), and optionally push images to a public registry (Docker Hub).

---

## **Lab Steps**

### **Part 1: Installing Docker**

1. **Download Docker Desktop.**
   - Visit [Docker Desktop](https://www.docker.com/products/docker-desktop) and download it for your OS (Windows, macOS, or Linux).

2. **Install Docker Desktop.**
   - Run the installer and follow on-screen instructions.

3. **Start Docker Desktop.**
   - Once installed, open Docker Desktop.
   - Confirm it’s running (Docker icon in your system tray/taskbar).

4. **Verify Docker installation.**
   - In a terminal or command prompt:
     ```bash
     docker --version
     ```
   - Confirm Docker is installed and you see a version number.

---

### **Part 2: Preparing the Microservices**

5. **Ensure `UserService` and `OrderService` are ready.**
   - Each microservice is a **Spring Boot 3.4.1** app with a simple controller.

6. **Add a Dockerfile to `UserService`.**
   - In the **root directory** of `UserService`, create `Dockerfile`:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     VOLUME /tmp
     ARG JAR_FILE=target/user-service-0.0.1-SNAPSHOT.jar
     COPY ${JAR_FILE} app.jar
     ENTRYPOINT ["java", "-jar", "/app.jar"]
     ```
   - Adjust `JAR_FILE` if your artifact name differs.

7. **Add a Dockerfile to `OrderService`.**
   - In `OrderService` root:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     VOLUME /tmp
     ARG JAR_FILE=target/order-service-0.0.1-SNAPSHOT.jar
     COPY ${JAR_FILE} app.jar
     ENTRYPOINT ["java", "-jar", "/app.jar"]
     ```
   - Same pattern as `UserService`.

8. **Build JAR files for both services.**
   - In each service folder:
     ```bash
     ./mvnw clean package
     ```
   - Ensure `target/user-service-0.0.1-SNAPSHOT.jar` and `target/order-service-0.0.1-SNAPSHOT.jar` exist.

---

### **Part 3: Building Docker Images**

9. **Build the Docker image for `UserService`.**
   - From the `UserService` directory:
     ```bash
     docker build -t user-service:1.0 .
     ```
   - The `-t` flag names and tags the image (`user-service:1.0`).

10. **Build the Docker image for `OrderService`.**
    - From `OrderService`:
      ```bash
      docker build -t order-service:1.0 .
      ```

11. **Verify Docker images.**
    - Check local images:
      ```bash
      docker images
      ```
    - You should see `user-service` and `order-service` with tag `1.0`.

---

### **Part 4: Running the Containers**

12. **Run the `UserService` container.**
    - Start a detached container:
      ```bash
      docker run -d --name user-service -p 8081:8081 user-service:1.0
      ```
    - `-p 8081:8081` maps host port 8081 to the container’s port 8081.

13. **Run the `OrderService` container.**
    - Similarly:
      ```bash
      docker run -d --name order-service -p 8082:8082 order-service:1.0
      ```

14. **Verify running containers.**
    - List containers:
      ```bash
      docker ps
      ```
    - Both `user-service` and `order-service` should be running.

15. **Test the microservices.**
    - `UserService`: `http://localhost:8081/users`
    - `OrderService`: `http://localhost:8082/orders`
    - Confirm they respond as before.

---

### **Part 5: Managing Docker Images and Containers**

16. **Stop the containers.**
    - e.g.:
      ```bash
      docker stop user-service
      docker stop order-service
      ```
    - The containers are no longer running but still exist.

17. **Restart the containers.**
    - `docker start user-service`
    - `docker start order-service`

18. **Remove containers.**
    - To remove them entirely:
      ```bash
      docker rm user-service
      docker rm order-service
      ```

19. **Remove Docker images.**
    - If you no longer need them locally:
      ```bash
      docker rmi user-service:1.0
      docker rmi order-service:1.0
      ```

---

### **Part 6: Publishing Docker Images**

20. **Tag the images for Docker Hub (or another registry).**
    - For `UserService`:
      ```bash
      docker tag user-service:1.0 <your-dockerhub-username>/user-service:1.0
      ```
    - For `OrderService`:
      ```bash
      docker tag order-service:1.0 <your-dockerhub-username>/order-service:1.0
      ```

21. **Log in to Docker Hub.**
    ```bash
    docker login
    ```
    - Enter your Docker Hub credentials.

22. **Push the images to Docker Hub.**
    - For `UserService`:
      ```bash
      docker push <your-dockerhub-username>/user-service:1.0
      ```
    - For `OrderService`:
      ```bash
      docker push <your-dockerhub-username>/order-service:1.0
      ```

23. **Verify images on Docker Hub.**
    - In your Docker Hub account, you should see both `user-service` and `order-service` repos with `1.0` tags.

---

## **Optional Exercises**

1. **Docker Compose for Multi-Container Deployment.**
   - Write a `docker-compose.yml` that runs both services together.
   - e.g.:
     ```yaml
     version: "3"
     services:
       user-service:
         image: user-service:1.0
         ports:
           - "8081:8081"
       order-service:
         image: order-service:1.0
         ports:
           - "8082:8082"
     ```

2. **Environment Variable Configuration.**
   - Modify `Dockerfile` or `docker-compose.yml` to pass environment variables (e.g., DB credentials).

3. **Integrate with CI/CD.**
   - Extend Jenkins or another CI pipeline to build and push these images automatically.

---

## **Conclusion**
You have successfully:
- **Containerized** your Spring Boot 3.4.1 microservices (`UserService` & `OrderService`) with Docker.
- **Built** Docker images locally and **ran** the containers.
- **Managed** container lifecycle (start, stop, remove) and optional **pushed** images to Docker Hub.

This approach ensures consistent and portable deployments across dev, test, and production environments!
