# **Lab 14: Set Up a CI Pipeline with Jenkins for Automated Testing and Deployment of Microservices (Spring Boot 3.4.1)**

## **Objective**
Learn how to install and configure **Jenkins** to automate the build, testing, and deployment process for **Spring Boot 3.4.1** microservices. You will create a CI pipeline that builds two microservices (`UserService` and `OrderService`) and optionally deploys them.

---

## **Lab Steps**

### **Part 1: Installing Jenkins**

1. **Install Java (Jenkins prerequisite).**
   - Confirm Java by running:
     ```bash
     java -version
     ```
   - If not found, install **JDK 17** (e.g., from [Adoptium](https://adoptium.net/)).
   - Verify installation:
     ```bash
     java -version
     ```

2. **Download Jenkins.**
   - Visit [Jenkins Downloads](https://www.jenkins.io/download/) and get the installer for your OS (Windows, macOS, or Linux).

3. **Install Jenkins.**
   - Run the installer and follow the wizard:
     - **Linux**: Use your package manager or `.deb`/`.rpm` packages.
     - **Windows**: Choose “**Run Jenkins as a Service**” option.
     - **macOS**: Use the `.pkg` installer.

4. **Start Jenkins.**
   - Jenkins usually starts on **port 8080**.
   - **Linux/macOS**:
     ```bash
     sudo systemctl start jenkins
     ```
   - **Windows**: Start it from **Services**.

5. **Verify Jenkins is running.**
   - Go to:
     ```
     http://localhost:8080
     ```

6. **Unlock Jenkins.**
   - Follow the prompt to retrieve the initial admin password:
     - **Linux/macOS**:
       ```bash
       sudo cat /var/lib/jenkins/secrets/initialAdminPassword
       ```
     - **Windows**:
       ```cmd
       type C:\Jenkins\secrets\initialAdminPassword
       ```
   - Copy/paste it into Jenkins.

7. **Install suggested plugins.**
   - Jenkins will prompt you to install **Suggested plugins**. Do so.

8. **Create an admin user.**
   - Finalize the setup by creating an admin account.

---

### **Part 2: Preparing the Microservices**

9. **Ensure `UserService` and `OrderService` are in Git repositories.**
   - Each microservice (`UserService`, `OrderService`) is a **Spring Boot 3.4.1** project.
   - Both are pushed to a hosting platform like **GitHub**.

10. **Configure `pom.xml` for Jenkins.**
    - In each microservice’s `pom.xml`, ensure the **Surefire plugin** is present for running tests:
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
    - This ensures Jenkins can run `mvn test` or `mvn clean install`.

---

### **Part 3: Configuring Jenkins for CI**

11. **Create a Jenkins job for `UserService`.**
    - From the Jenkins dashboard, click **New Item**.
    - Name it `UserService-CI`, choose **Freestyle project**.
    - Click **OK**.

12. **Set up the Git repository in Jenkins.**
    - Under **Source Code Management**, select **Git**.
    - Enter the repository URL for `UserService`.

13. **Add a Maven build step.**
    - In the **Build** section, add **Invoke top-level Maven targets**.
    - Goals:
      ```
      clean install
      ```

14. **Save and run the job.**
    - Click **Build Now**.
    - Confirm the build passes.

15. **Create a job for `OrderService`.**
    - Same approach: name it `OrderService-CI`, configure the Git repo, and `clean install`.

---

### **Part 4: Creating a Jenkins Pipeline**

16. **Install the Pipeline plugin if not installed.**
    - Go to **Manage Jenkins** → **Manage Plugins**.
    - Search **Pipeline** and install it.

17. **Create a new pipeline job.**
    - Go to dashboard → **New Item**.
    - Select **Pipeline**, name it `Microservices-CI-Pipeline`.
    - Click **OK**.

18. **Write the pipeline script.**
    - Under **Pipeline** → **Pipeline script**:
      ```groovy
      pipeline {
          agent any
          stages {
              stage('Build UserService') {
                  steps {
                      script {
                          // Clone UserService repo
                          git url: '<UserService Git URL>'
                          dir('user-service') {
                              sh './mvnw clean install'
                          }
                      }
                  }
              }
              stage('Build OrderService') {
                  steps {
                      script {
                          // Clone OrderService repo
                          git url: '<OrderService Git URL>'
                          dir('order-service') {
                              sh './mvnw clean install'
                          }
                      }
                  }
              }
          }
      }
      ```
    - Replace `<UserService Git URL>` and `<OrderService Git URL>` with actual repositories.
    - On **Windows** agents, use `bat` instead of `sh`.

19. **Save and run the pipeline.**
    - Click **Build Now**.
    - Confirm both microservices build successfully in one pipeline run.

---

### **Part 5: Deployment (Optional)**

20. **Add a deployment stage.**
    - Example:
      ```groovy
      stage('Deploy to Server') {
          steps {
              script {
                  sh 'scp user-service/target/*.jar user@server:/path/to/deploy/'
                  sh 'scp order-service/target/*.jar user@server:/path/to/deploy/'
              }
          }
      }
      ```
    - This is just a placeholder. You can also push Docker images, etc.

21. **Configure automated triggers.**
    - In your GitHub (or other Git server), set up **webhooks** to trigger Jenkins on each push.

---

## **Optional Exercises**

1. **Integrate automated tests.**
   - Add a pipeline stage to run integration tests or Docker-based tests.

2. **Set up a multi-branch pipeline.**
   - Use the **Multibranch Pipeline Plugin** to automatically create pipeline jobs for each branch.

3. **Add Docker integration.**
   - Build Docker images in your pipeline, then push them to Docker Hub or another registry.

4. **Monitor Jenkins builds.**
   - Use plugins like **Build Monitor View** or **Blue Ocean** for improved visualization of job statuses.

---

## **Conclusion**
By completing this lab, you have:
- **Installed and configured Jenkins** on port **8080**.
- **Set up Freestyle jobs** for each microservice (UserService, OrderService).
- **Created a pipeline** that clones, builds, and optionally deploys both microservices in a single run.
- **Learned** how Jenkins can automate your entire CI flow, ensuring code changes are consistently tested and deployed. Enjoy building more advanced CI/CD pipelines!
