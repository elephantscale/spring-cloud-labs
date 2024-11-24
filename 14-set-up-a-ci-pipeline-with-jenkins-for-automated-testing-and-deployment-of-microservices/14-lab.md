# **Lab 14: Set Up a CI Pipeline with Jenkins for Automated Testing and Deployment of Microservices**

## **Objective**
Learn how to install and configure Jenkins on Linux to automate the build, testing, and deployment process for Spring Boot microservices. Create a Jenkins pipeline that builds and deploys two microservices: `UserService` and `OrderService`.

---

## **Lab Steps**

### **Part 1: Installing Jenkins on Linux**

1. **Update system packages.**
   - Open a terminal and run:
     ```bash
     sudo apt update && sudo apt upgrade -y
     ```

2. **Install Java (required for Jenkins).**
   - Verify if Java is installed:
     ```bash
     java -version
     ```
   - If Java is not installed, install OpenJDK:
     ```bash
     sudo apt install openjdk-17-jdk -y
     ```
   - Verify the installation:
     ```bash
     java -version
     ```

3. **Add the Jenkins repository key.**
   - Run the following commands to add the Jenkins repository and key:
     ```bash
     curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
     /usr/share/keyrings/jenkins-keyring.asc > /dev/null
     echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
     https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
     /etc/apt/sources.list.d/jenkins.list > /dev/null
     ```

4. **Install Jenkins.**
   - Update the package list:
     ```bash
     sudo apt update
     ```
   - Install Jenkins:
     ```bash
     sudo apt install jenkins -y
     ```

5. **Start Jenkins.**
   - Start the Jenkins service:
     ```bash
     sudo systemctl start jenkins
     ```
   - Enable Jenkins to start on boot:
     ```bash
     sudo systemctl enable jenkins
     ```

6. **Verify Jenkins is running.**
   - Check the status of Jenkins:
     ```bash
     sudo systemctl status jenkins
     ```
   - Open a browser and navigate to:
     ```
     http://localhost:8080
     ```

7. **Unlock Jenkins.**
   - Retrieve the initial admin password:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Copy the password and paste it into the Jenkins setup page.

8. **Install suggested plugins.**
   - Follow the Jenkins setup wizard and select **Install suggested plugins**.

9. **Create an admin user.**
   - Create a new admin user with your credentials and complete the setup.

---

### **Part 2: Preparing the Microservices**

10. **Ensure `UserService` and `OrderService` are ready.**
    - Ensure the two microservices (`UserService` and `OrderService`) are in separate Git repositories.
    - Push the projects to a Git hosting platform (e.g., GitHub).

11. **Configure `pom.xml` for Jenkins integration.**
    - Ensure the `pom.xml` files of both microservices include a `surefire` plugin for running tests:
      ```xml
      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-surefire-plugin</artifactId>
                  <version>3.0.0</version>
              </plugin>
          </plugins>
      </build>
      ```

---

### **Part 3: Configuring Jenkins for CI**

12. **Create a Jenkins job for `UserService`.**
    - Navigate to the Jenkins dashboard and click **New Item**.
    - Enter the name `UserService-CI` and select **Freestyle Project**.
    - Click **OK**.

13. **Configure the Git repository for `UserService`.**
    - In the project configuration, go to the **Source Code Management** section.
    - Select **Git** and add the repository URL for `UserService`.

14. **Add a build step for Maven.**
    - In the **Build** section, click **Add build step** and select **Invoke top-level Maven targets**.
    - Set the **Goals** to:
      ```bash
      clean install
      ```

15. **Save and build the job.**
    - Click **Save** and then click **Build Now**.
    - Verify that the build succeeds.

16. **Create a Jenkins job for `OrderService`.**
    - Repeat steps 12–15 for the `OrderService` microservice.

---

### **Part 4: Creating a Jenkins Pipeline**

17. **Install the Pipeline plugin.**
    - Navigate to **Manage Jenkins > Manage Plugins**.
    - Search for **Pipeline** and install it.

18. **Create a new pipeline job.**
    - Go to the Jenkins dashboard, click **New Item**, and select **Pipeline**.
    - Name it `Microservices-CI-Pipeline` and click **OK**.

19. **Define the pipeline script.**
    - In the **Pipeline** section, select **Pipeline script**.
    - Add the following script:
      ```groovy
      pipeline {
          agent any
          stages {
              stage('Build UserService') {
                  steps {
                      sh 'git clone <UserService Git Repository>'
                      dir('user-service') {
                          sh './mvnw clean install'
                      }
                  }
              }
              stage('Build OrderService') {
                  steps {
                      sh 'git clone <OrderService Git Repository>'
                      dir('order-service') {
                          sh './mvnw clean install'
                      }
                  }
              }
          }
      }
      ```
      - Replace `<UserService Git Repository>` and `<OrderService Git Repository>` with the actual repository URLs.

20. **Save and run the pipeline.**
    - Save the pipeline and click **Build Now**.
    - Verify that both services are built successfully.

---

### **Part 5: Deployment (Optional)**

21. **Deploy artifacts to a server.**
    - Add a new pipeline stage for deployment. For example:
      ```groovy
      stage('Deploy to Server') {
          steps {
              sh 'scp target/*.jar user@server:/path/to/deployment/'
          }
      }
      ```

22. **Automate deployment triggers.**
    - Configure webhooks in your Git repositories to trigger builds automatically when code is pushed.

---

### **Optional Exercises (20 mins)**

1. **Add test automation to the pipeline.**
   - Add a pipeline stage to execute integration tests before deployment.

2. **Set up a multi-branch pipeline.**
   - Use the **Pipeline Multibranch Plugin** to automatically detect branches and execute CI/CD pipelines for them.

3. **Integrate with Docker.**
   - Build Docker images for the microservices as part of the CI pipeline and push them to a Docker registry.

---
