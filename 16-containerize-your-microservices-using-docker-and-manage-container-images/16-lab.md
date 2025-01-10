# **Lab 16: Containerize Your Microservices Using Docker and Manage Container Images**

## **Objective**
Learn how to containerize Spring Boot microservices using Docker. Build Docker images for `UserService` and `OrderService`, manage container configurations, and run them using Docker.

---

## **Lab Steps**

### **Part 1: Installing Docker**

1. **Download Docker Desktop.**
   - Visit [Docker Desktop](https://www.docker.com/products/docker-desktop) and download the installer for your operating system (Windows, macOS, or Linux).

2. **Install Docker Desktop.**
   - Run the installer and follow the setup instructions.

3. **Start Docker Desktop.**
   - Open Docker Desktop after installation.
   - Ensure Docker is running (look for the Docker icon in the system tray or taskbar).

4. **Verify Docker installation.**
   - Open a terminal or command prompt and run:
     ```bash
     docker --version
     ```
   - Confirm that Docker is installed and the version is displayed.

---

### **Part 2: Preparing the Microservices**

5. **Ensure `UserService` and `OrderService` are ready.**
   - Make sure the microservices (`UserService` and `OrderService`) are functional and available as standalone Spring Boot projects.

6. **Add a Dockerfile to `UserService`.**
   - Create a `Dockerfile` in the root directory of the `UserService` project:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     VOLUME /tmp
     ARG JAR_FILE=target/user-service-0.0.1-SNAPSHOT.jar
     COPY ${JAR_FILE} app.jar
     ENTRYPOINT ["java", "-jar", "/app.jar"]
     ```

7. **Add a Dockerfile to `OrderService`.**
   - Create a `Dockerfile` in the root directory of the `OrderService` project:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     VOLUME /tmp
     ARG JAR_FILE=target/order-service-0.0.1-SNAPSHOT.jar
     COPY ${JAR_FILE} app.jar
     ENTRYPOINT ["java", "-jar", "/app.jar"]
     ```

8. **Build the JAR files for both services.**
   - Navigate to each project directory and run:
     ```bash
     ./mvnw clean package
     ```

---

### **Part 3: Building Docker Images**

9. **Build the Docker image for `UserService`.**
    - Run the following command from the `UserService` directory:
      ```bash
      docker build -t user-service:1.0 .
      ```

10. **Build the Docker image for `OrderService`.**
    - Run the following command from the `OrderService` directory:
      ```bash
      docker build -t order-service:1.0 .
      ```

11. **Verify Docker images.**
    - Check the built images using:
      ```bash
      docker images
      ```

---

### **Part 4: Running the Containers**

12. **Run the `UserService` container.**
    - Start a container for `UserService`:
      ```bash
      docker run -d --name user-service -p 8081:8081 user-service:1.0
      ```

13. **Run the `OrderService` container.**
    - Start a container for `OrderService`:
      ```bash
      docker run -d --name order-service -p 8082:8082 order-service:1.0
      ```

14. **Verify running containers.**
    - List all running containers:
      ```bash
      docker ps
      ```

15. **Test the microservices.**
    - Access `UserService`:
      ```
      http://localhost:8081/users
      ```
    - Access `OrderService`:
      ```
      http://localhost:8082/orders
      ```

---

### **Part 5: Managing Docker Images and Containers**

16. **Stop the containers.**
    - Stop both services:
      ```bash
      docker stop user-service
      docker stop order-service
      ```

17. **Restart the containers.**
    - Start the containers again:
      ```bash
      docker start user-service
      docker start order-service
      ```

18. **Remove containers.**
    - Remove the stopped containers:
      ```bash
      docker rm user-service
      docker rm order-service
      ```

19. **Remove Docker images.**
    - Delete the Docker images:
      ```bash
      docker rmi user-service:1.0
      docker rmi order-service:1.0
      ```

---

### **Part 6: Publishing Docker Images**

20. **Tag the images for a Docker registry.**
    - Tag the `UserService` image:
      ```bash
      docker tag user-service:1.0 <your-dockerhub-username>/user-service:1.0
      ```
    - Tag the `OrderService` image:
      ```bash
      docker tag order-service:1.0 <your-dockerhub-username>/order-service:1.0
      ```

21. **Log in to Docker Hub.**
    - Authenticate with Docker Hub:
      ```bash
      docker login
      ```

22. **Push the images to Docker Hub.**
    - Push `UserService`:
      ```bash
      docker push <your-dockerhub-username>/user-service:1.0
      ```
    - Push `OrderService`:
      ```bash
      docker push <your-dockerhub-username>/order-service:1.0
      ```

23. **Verify images on Docker Hub.**
    - Open Docker Hub in a browser and confirm that both images are available in your repository.

---

### **Optional Exercises (20 mins)**

1. **Docker Compose for Multi-Container Deployment.**
   - Write a `docker-compose.yml` file to deploy both `UserService` and `OrderService` together.

2. **Environment Variable Configuration.**
   - Modify the `Dockerfile` to pass environment variables to the Spring Boot applications.

3. **Integrate with CI/CD.**
   - Configure Jenkins to build and push Docker images automatically after successful builds.

---
