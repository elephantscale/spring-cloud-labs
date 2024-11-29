# **Lab 14: Set Up a CI Pipeline with Jenkins for Automated Testing and Deployment of Microservices**

## **Objective**
Learn how to install and configure Jenkins on Windows to automate the build, testing, and deployment process for Spring Boot microservices. Create a Jenkins pipeline that builds and deploys two microservices: `UserService` and `OrderService`.

---

## **Lab Steps**

### **Part 1: Installing Jenkins on Windows**

1. **Install Java (required for Jenkins).**
   - Open a Command Prompt and check if Java is installed:
     ```cmd
     java -version
     ```
   - If Java is not installed:
     - Download JDK 17 from [https://adoptium.net/](https://adoptium.net/).
     - Install it by following the setup instructions.
   - Verify the installation:
     ```cmd
     java -version
     ```

2. **Download Jenkins.**
   - Visit [https://www.jenkins.io/download/](https://www.jenkins.io/download/) and download the **Windows Installer**.

3. **Install Jenkins.**
   - Run the installer and follow the setup instructions.
   - During installation:
     - Choose the installation folder (e.g., `C:\Jenkins`).
     - Select **Run Jenkins as a Service**.

4. **Start Jenkins.**
   - Jenkins should start automatically after installation. If not, open **Services**, find **Jenkins**, and start the service manually.

5. **Verify Jenkins is running.**
   - Open a browser and navigate to:
     ```
     http://localhost:8080
     ```

6. **Unlock Jenkins.**
   - Retrieve the initial admin password:
     ```cmd
     type C:\Jenkins\secrets\initialAdminPassword
     ```
   - Copy the password and paste it into the Jenkins setup page.

7. **Install suggested plugins.**
   - Follow the Jenkins setup wizard and select **Install suggested plugins**.

8. **Create an admin user.**
   - Create a new admin user with your credentials and complete the setup.

---

### **Part 2: Preparing the Microservices**

9. **Ensure `UserService` and `OrderService` are ready.**
   - Ensure the two microservices (`UserService` and `OrderService`) are in separate Git repositories.
   - Push the projects to a Git hosting platform (e.g., GitHub).

10. **Configure `pom.xml` for Jenkins integration.**
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

11. **Create a Jenkins job for `UserService`.**
    - Navigate to the Jenkins dashboard and click **New Item**.
    - Enter the name `UserService-CI` and select **Freestyle Project**.
    - Click **OK**.

12. **Configure the Git repository for `UserService`.**
    - In the project configuration, go to the **Source Code Management** section.
    - Select **Git** and add the repository URL for `UserService`.

13. **Add a build step for Maven.**
    - In the **Build** section, click **Add build step** and select **Invoke top-level Maven targets**.
    - Set the **Goals** to:
      ```cmd
      clean install
      ```

14. **Save and build the job.**
    - Click **Save** and then click **Build Now**.
    - Verify that the build succeeds.

15. **Create a Jenkins job for `OrderService`.**
    - Repeat steps 11–14 for the `OrderService` microservice.

---

### **Part 4: Creating a Jenkins Pipeline**

16. **Install the Pipeline plugin.**
    - Navigate to **Manage Jenkins > Manage Plugins**.
    - Search for **Pipeline** and install it.

17. **Create a new pipeline job.**
    - Go to the Jenkins dashboard, click **New Item**, and select **Pipeline**.
    - Name it `Microservices-CI-Pipeline` and click **OK**.

18. **Define the pipeline script.**
    - In the **Pipeline** section, select **Pipeline script**.
    - Add the following script:
      ```groovy
      pipeline {
          agent any
          stages {
              stage('Build UserService') {
                  steps {
                      bat 'git clone <UserService Git Repository>'
                      dir('user-service') {
                          bat './mvnw clean install'
                      }
                  }
              }
              stage('Build OrderService') {
                  steps {
                      bat 'git clone <OrderService Git Repository>'
                      dir('order-service') {
                          bat './mvnw clean install'
                      }
                  }
              }
          }
      }
      ```
      - Replace `<UserService Git Repository>` and `<OrderService Git Repository>` with the actual repository URLs.

19. **Save and run the pipeline.**
    - Save the pipeline and click **Build Now**.
    - Verify that both services are built successfully.

---

### **Part 5: Deployment (Optional)**

20. **Deploy artifacts to a server.**
    - Add a new pipeline stage for deployment. For example:
      ```groovy
      stage('Deploy to Server') {
          steps {
              bat 'copy target/*.jar \\\\<server>\\deployment\\'
          }
      }
      ```

21. **Automate deployment triggers.**
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
