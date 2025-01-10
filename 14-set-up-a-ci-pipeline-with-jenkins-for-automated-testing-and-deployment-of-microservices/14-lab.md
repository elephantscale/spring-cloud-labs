# **Lab 14: Set Up a CI Pipeline with Jenkins for Automated Testing and Deployment of Microservices**

## **Objective**
Learn how to install and configure Jenkins to automate the build, testing, and deployment process for Spring Boot microservices. Create a Jenkins pipeline that builds and deploys two microservices: `UserService` and `OrderService`.

---

## **Lab Steps**

### **Part 1: Installing Jenkins**

1. **Install Java (required for Jenkins).**
   - Open a terminal or Command Prompt and check if Java is installed:
     ```bash
     java -version
     ```
   - If Java is not installed:
     - Download JDK 17 from [Adoptium](https://adoptium.net/).
     - Install it by following the setup instructions.
   - Verify the installation:
     ```bash
     java -version
     ```

2. **Download Jenkins.**
   - Visit [Jenkins Downloads](https://www.jenkins.io/download/) and download the installer for your operating system.

3. **Install Jenkins.**
   - Run the installer and follow the setup instructions:
     - For Linux: Use the package manager for your distribution.
     - For Windows: Choose **Run Jenkins as a Service** during installation.
     - For macOS: Use the `.pkg` installer.

4. **Start Jenkins.**
   - If Jenkins does not start automatically:
     - **Linux/macOS**: Run:
       ```bash
       sudo systemctl start jenkins
       ```
     - **Windows**: Start Jenkins from **Services**.

5. **Verify Jenkins is running.**
   - Open a browser and navigate to:
     ```
     http://localhost:8080
     ```

6. **Unlock Jenkins.**
   - Retrieve the initial admin password:
     - **Linux/macOS**:
       ```bash
       sudo cat /var/lib/jenkins/secrets/initialAdminPassword
       ```
     - **Windows**:
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
   - Ensure the two microservices are in separate Git repositories.
   - Push the projects to a Git hosting platform (e.g., GitHub, GitLab, or Bitbucket).

10. **Configure `pom.xml` for Jenkins integration.**
    - Ensure the `pom.xml` files of both microservices include the `surefire` plugin for running tests:
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
      ```
      clean install
      ```

14. **Save and build the job.**
    - Click **Save** and then **Build Now**.
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
                      script {
                          if (isUnix()) {
                              sh 'git clone <UserService Git Repository>'
                              dir('user-service') {
                                  sh './mvnw clean install'
                              }
                          } else {
                              bat 'git clone <UserService Git Repository>'
                              dir('user-service') {
                                  bat './mvnw clean install'
                              }
                          }
                      }
                  }
              }
              stage('Build OrderService') {
                  steps {
                      script {
                          if (isUnix()) {
                              sh 'git clone <OrderService Git Repository>'
                              dir('order-service') {
                                  sh './mvnw clean install'
                              }
                          } else {
                              bat 'git clone <OrderService Git Repository>'
                              dir('order-service') {
                                  bat './mvnw clean install'
                              }
                          }
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
              script {
                  if (isUnix()) {
                      sh 'scp target/*.jar user@server:/path/to/deployment/'
                  } else {
                      bat 'copy target\\*.jar \\\\<server>\\deployment\\'
                  }
              }
          }
      }
      ```

21. **Automate deployment triggers.**
    - Configure webhooks in your Git repositories to trigger builds automatically when code is pushed.

---

### **Optional Exercises**

1. **Add test automation to the pipeline.**
   - Add a pipeline stage to execute integration tests before deployment.

2. **Set up a multi-branch pipeline.**
   - Use the **Pipeline Multibranch Plugin** to automatically detect branches and execute CI/CD pipelines for them.

3. **Integrate with Docker.**
   - Build Docker images for the microservices as part of the CI pipeline and push them to a Docker registry.

---
