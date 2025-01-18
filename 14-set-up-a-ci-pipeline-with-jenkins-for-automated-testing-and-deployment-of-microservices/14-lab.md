# **Lab 14: Set Up a CI Pipeline with Jenkins for Automated Testing and Deployment of Microservices**

## **Objective**
Learn how to install and configure Jenkins to automate the build, testing, and deployment process for Spring Boot microservices. Create a Jenkins pipeline that builds and deploys two microservices: `UserService` and `OrderService`.

---

## **Lab Steps**

### **Part 1: Installing Jenkins**

1. **Install Java (required for Jenkins).**
   - Check if Java is installed:
     ```bash
     java -version
     ```
   - If not installed:
     - Download JDK 17 from [Adoptium](https://adoptium.net/).
     - Follow the installation instructions.
   - Verify Java installation:
     ```bash
     java -version
     ```

2. **Download Jenkins.**
   - Visit [Jenkins Downloads](https://www.jenkins.io/download/) and download the installer for your operating system.

3. **Install Jenkins.**
   - Run the installer and follow the setup instructions:
     - For Linux: Use your package manager (e.g., `apt`, `yum`).
     - For Windows: Choose the **Run Jenkins as a Service** option.
     - For macOS: Use the `.pkg` installer.

4. **Start Jenkins.**
   - Start Jenkins:
     - **Linux/macOS**:
       ```bash
       sudo systemctl start jenkins
       ```
     - **Windows**: Start Jenkins from **Services**.

5. **Verify Jenkins is running.**
   - Open your browser and navigate to:
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
   - Paste the password into the Jenkins setup page.

7. **Install suggested plugins.**
   - Follow the Jenkins setup wizard and select **Install suggested plugins**.

8. **Create an admin user.**
   - Create an admin account and complete the setup.

---

### **Part 2: Preparing the Microservices**

9. **Ensure `UserService` and `OrderService` are ready.**
   - Ensure both microservices are in separate Git repositories.
   - Push the projects to a Git hosting platform like GitHub.

10. **Configure `pom.xml` for Jenkins integration.**
    - Ensure both `pom.xml` files include the `surefire` plugin:
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
    - Go to the Jenkins dashboard and click **New Item**.
    - Enter the name `UserService-CI` and select **Freestyle Project**.
    - Click **OK**.

12. **Set up the Git repository for `UserService`.**
    - In **Source Code Management**, choose **Git** and enter the repository URL.

13. **Add a Maven build step.**
    - In the **Build** section, add **Invoke top-level Maven targets**.
    - Set **Goals** to:
      ```
      clean install
      ```

14. **Save and run the build.**
    - Save the job and click **Build Now**.
    - Verify the build completes successfully.

15. **Repeat for `OrderService`.**
    - Create another Jenkins job, `OrderService-CI`, and configure it similarly.

---

### **Part 4: Creating a Jenkins Pipeline**

16. **Install the Pipeline plugin.**
    - Navigate to **Manage Jenkins > Manage Plugins**.
    - Search for **Pipeline** and install it.

17. **Create a new pipeline job.**
    - Go to the Jenkins dashboard and click **New Item**.
    - Select **Pipeline** and name it `Microservices-CI-Pipeline`.
    - Click **OK**.

18. **Write the pipeline script.**
    - In the **Pipeline** section, select **Pipeline script**.
    - Add the following script:
      ```groovy
      pipeline {
          agent any
          stages {
              stage('Build UserService') {
                  steps {
                      script {
                          git url: '<UserService Git Repository>'
                          dir('user-service') {
                              sh './mvnw clean install'
                          }
                      }
                  }
              }
              stage('Build OrderService') {
                  steps {
                      script {
                          git url: '<OrderService Git Repository>'
                          dir('order-service') {
                              sh './mvnw clean install'
                          }
                      }
                  }
              }
          }
      }
      ```
      - Replace `<UserService Git Repository>` and `<OrderService Git Repository>` with the actual URLs.

19. **Save and run the pipeline.**
    - Save the job and click **Build Now**.
    - Verify that both microservices are built successfully.

---

### **Part 5: Deployment (Optional)**

20. **Add deployment steps.**
    - Add a deployment stage to the pipeline script:
      ```groovy
      stage('Deploy to Server') {
          steps {
              script {
                  sh 'scp target/*.jar user@server:/path/to/deployment/'
              }
          }
      }
      ```

21. **Configure automated triggers.**
    - Set up webhooks in your Git repositories to trigger the pipeline on code changes.

---

### **Optional Exercises**

1. **Integrate automated tests.**
   - Add a stage to execute integration tests before deployment.

2. **Set up a multi-branch pipeline.**
   - Use the **Pipeline Multibranch Plugin** to detect branches automatically.

3. **Add Docker integration.**
   - Build Docker images in the pipeline and push them to a registry.

4. **Monitor Jenkins builds.**
   - Install and configure plugins like **Build Monitor View** or **Blue Ocean** for visualization.
