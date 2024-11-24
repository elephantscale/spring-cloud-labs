# **Lab 16: Containerize Your Microservices Using Docker and Manage Container Images**

## **Objective**
Learn how to containerize Spring Boot microservices using Docker. Build Docker images for `UserService` and `OrderService`, manage container configurations, and run them using Docker.

---

## **Lab Steps**

### **Part 1: Installing Docker on Linux**

1. **Update system packages.**
   - Open a terminal and run:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

2. **Install Docker.**
   - Run the following commands to install Docker:
     ```bash
     sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
     echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
     sudo apt update
     sudo apt install docker-ce -y
     ```

3. **Verify Docker installation.**
   - Run the following command:
     ```bash
     docker --version
     ```

4. **Enable Docker service.**
   - Start and enable Docker:
     ```bash
     sudo systemctl start docker
     sudo systemctl enable docker
     ```

5. **Add your user to the Docker group (optional).**
   - Run:
     ```bash
     sudo usermod -aG docker $USER
     ```
   - Log out and back in for the changes to take effect.

---

### **Part 2: Preparing the Microservices**

6. **Ensure `UserService` and `OrderService` are ready.**
   - Make sure the microservices (`UserService` and `OrderService`) are functional and available as standalone Spring Boot projects.

7. **Add a Dockerfile to `UserService`.**
   - Create a `Dockerfile` in the root directory of the `UserService` project:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     VOLUME /tmp
     ARG JAR_FILE=target/user-service-0.0.1-SNAPSHOT.jar
     COPY ${JAR_FILE} app.jar
     ENTRYPOINT ["java", "-jar", "/app.jar"]
     ```

8. **Add a Dockerfile to `OrderService`.**
   - Create a `Dockerfile` in the root directory of the `OrderService` project:
     ```dockerfile
     FROM openjdk:17-jdk-slim
     VOLUME /tmp
     ARG JAR_FILE=target/order-service-0.0.1-SNAPSHOT.jar
     COPY ${JAR_FILE} app.jar
     ENTRYPOINT ["java", "-jar", "/app.jar"]
     ```

9. **Build the JAR files for both services.**
   - Navigate to each project directory and run:
     ```bash
     ./mvnw clean package
     ```

---

### **Part 3: Building Docker Images**

10. **Build the Docker image for `UserService`.**
    - Run the following command from the `UserService` directory:
      ```bash
      docker build -t user-service:1.0 .
      ```

11. **Build the Docker image for `OrderService`.**
    - Run the following command from the `OrderService` directory:
      ```bash
      docker build -t order-service:1.0 .
      ```

12. **Verify Docker images.**
    - Check the built images using:
      ```bash
      docker images
      ```

---

### **Part 4: Running the Containers**

13. **Run the `UserService` container.**
    - Start a container for `UserService`:
      ```bash
      docker run -d --name user-service -p 8081:8081 user-service:1.0
      ```

14. **Run the `OrderService` container.**
    - Start a container for `OrderService`:
      ```bash
      docker run -d --name order-service -p 8082:8082 order-service:1.0
      ```

15. **Verify running containers.**
    - List all running containers:
      ```bash
      docker ps
      ```

16. **Test the microservices.**
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

17. **Stop the containers.**
    - Stop both services:
      ```bash
      docker stop user-service
      docker stop order-service
      ```

18. **Restart the containers.**
    - Start the containers again:
      ```bash
      docker start user-service
      docker start order-service
      ```

19. **Remove containers.**
    - Remove the stopped containers:
      ```bash
      docker rm user-service
      docker rm order-service
      ```

20. **Remove Docker images.**
    - Delete the Docker images:
      ```bash
      docker rmi user-service:1.0
      docker rmi order-service:1.0
      ```

---

### **Part 6: Publishing Docker Images**

21. **Tag the images for a Docker registry.**
    - Tag the `UserService` image:
      ```bash
      docker tag user-service:1.0 <your-dockerhub-username>/user-service:1.0
      ```
    - Tag the `OrderService` image:
      ```bash
      docker tag order-service:1.0 <your-dockerhub-username>/order-service:1.0
      ```

22. **Log in to Docker Hub.**
    - Authenticate with Docker Hub:
      ```bash
      docker login
      ```

23. **Push the images to Docker Hub.**
    - Push `UserService`:
      ```bash
      docker push <your-dockerhub-username>/user-service:1.0
      ```
    - Push `OrderService`:
      ```bash
      docker push <your-dockerhub-username>/order-service:1.0
      ```

24. **Verify images on Docker Hub.**
    - Open Docker Hub in a browser and confirm that both images are available in your repository.

---

### **Optional Exercises (20 mins)**

1. **Docker Compose for Multi-Container Deployment.**
   - Write a `docker-compose.yml` file to deploy both `UserService` and `OrderService` together.

2. **Environment Variable Configuration.**
   - Modify the `Dockerfile` to pass environment variables to the Spring Boot applications.

3. **Integrate with CI/CD.**
   - Configure Jenkins to build and push Docker images automatically after successful builds.

