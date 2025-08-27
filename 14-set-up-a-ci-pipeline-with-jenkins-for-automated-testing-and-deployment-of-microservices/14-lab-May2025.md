# **Lab 14: Set Up a CI Pipeline with Jenkins for Automated Testing and Deployment of Microservices (Spring Boot 3.4.5)**

## **Objective**
Learn how to install and configure **Jenkins** to automate the build, testing, and deployment process for **Spring Boot 3.4.5** microservices. You will create a CI pipeline that builds two microservices (`user-service` and `order-service`) and optionally deploys them.

---

## **GitHub CLI Setup — do this once**

> We’ll install the GitHub CLI and authenticate so you never have to copy a token manually.

| Step | What to do (PowerShell **Administrator**) | Expected Result |
|------|-------------------------------------------|-----------------|
| 1 | **Install GitHub CLI**<br>`winget install --id GitHub.cli` | *Successfully installed GitHub CLI* |
| 2 | **Verify installation**<br>`gh --version` | Prints e.g. `gh version 2.51.0` |
| 3 | **Authenticate**<br>`gh auth login` → choose **GitHub.com** → **HTTPS** → browser opens, sign in, click **Authorize** | Terminal shows `✓ Logged in as <your‑username>` |

---

## **Lab Steps**

### **Part 1: Installing Jenkins**

1. **Install Java (Jenkins prerequisite).**
   - Confirm Java by running:
     ```cmd
     java -version
     ```
     ✅ Expected output:
     ```
     openjdk version "17.x.x"
     ```
   - If not found, install **JDK 17** from [Adoptium](https://adoptium.net/).

2. **Download Jenkins.**
   - Visit [Jenkins Downloads](https://www.jenkins.io/download/) and get the installer for **Windows**.

3. **Install Jenkins.**
   - Run the installer and choose “**Run Jenkins as a Service**”.

4. **Start Jenkins.**
   - Open **Services** from the Start Menu and start the Jenkins service.

5. **Verify Jenkins is running.**
   - Open browser and go to:
     ```
     http://localhost:8080
     ```

6. **Unlock Jenkins.**
   - Retrieve the initial admin password:
     ```cmd
     type C:\Jenkins\secrets\initialAdminPassword
     ```
   - Copy/paste it into Jenkins.

7. **Install suggested plugins.**
   - Jenkins will prompt you to install **Suggested plugins**. Do so.

8. **Create an admin user.**
   - Finalize the setup by creating an admin account.

> **What is Jenkins?**  
> Jenkins is an open-source automation server. In this lab, you’ll use it to automatically test and build your Spring Boot microservices when you push code.

---

## Part 2: Setting Up the Microservices (`order-service` & `user-service`)

**Create projects.**  
   Make sure you have extracted the starter files for **Lab 14**.
   - 📂 Place **studentCloud.zip** in `C:\` and **Extract All`.  
   - 📂 Create folder `C:\studentCloudLabs`.  
   - 📋 Copy **lab14** from `C:\studentCloud\starters` → paste into `C:\studentCloudLabs`.

   Once Lab 14 is copied, unzip the service folders:
   - 🗂️ Inside `lab14`, unzip service files (e.g., `order-service.zip` → `order-service`, `user-service.zip` → `user-service`).  
   - ✅ Verify structure, e.g.:  
     ```
     C:\studentCloudLabs
       └─ lab14
           ├─ order-service
           └─ user-service
     ```

**Open in IDE and update the `pom.xml` for the Producer (`order-service`).**  
   - Open **IntelliJ IDEA** (or VS Code) → **File → Open…** → select `C:\studentCloudLabs\lab14\order-service`.  
   - Wait for Maven/Gradle import to finish.  
   - Add the following dependencies to `order-service/pom.xml`:
   ```xml
      <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
   ```

**Open in IDE and update the `pom.xml` for the Producer (`user-service`).**  
   - Open **IntelliJ IDEA** (or VS Code) → **File → Open…** → select `C:\studentCloudLabs\lab14\user-service`.  
   - Wait for Maven/Gradle import to finish.  
   - Add the following dependencies to `user-service/pom.xml`:
   ```xml
      <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
   ```

> 💡 Tip: In IntelliJ, press **Alt + F12** (or click **Terminal**) inside each project to run build/test commands once dependencies finish indexing.

---

9. **Push services to GitHub.**

### **Create & push each microservice repo**

#### GitHub CLI — `user-service`

| Step | What to do | Expected Result |
|------|------------|-----------------|
| A | **Create the repo with a README**<br>`gh repo create user-service --public --add-readme --confirm` | Repo URL printed; remote **user-service** repo created |
| B | **Commit & push**<br>`git add .`<br>`git commit -m "Initial commit – user-service"`<br>`git push -u origin main` | Push completes; branch **main** on GitHub |

#### GitHub CLI — `order-service`

| Step | What to do | Expected Result |
|------|------------|-----------------|
| A | **Create the repo with a README**<br>`gh repo create order-service --public --add-readme --confirm` | Repo URL printed; remote **order-service** repo created |
| B | **Commit & push**<br>`git add .`<br>`git commit -m "Initial commit – order-service"`<br>`git push -u origin main` | Push completes; branch **main** on GitHub |

10. **Verify `pom.xml` contains Surefire plugin.**
📄 `user-service/pom.xml` and `order-service/pom.xml`:
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
💡 Note: Do not manually add versions for Spring Boot dependencies. Use Spring Initializr's managed versions.

---

### **Part 3: Configuring Jenkins for CI**

11. **Create a Jenkins job for `user-service`.**
    - From the Jenkins dashboard, click **New Item**.
    - Name it `user-service-CI`, choose **Freestyle project**, and click **OK**.

12. **Set up the Git repository in Jenkins.**
    - Under **Source Code Management**, select **Git**.
    - Under **Branch Specifier**, Enter exactly ***/main**
    - Enter the GitHub repository URL for `user-service`.

13. **Add a Maven build step.**
    - In the **Build** section, add **Invoke top-level Maven targets**.
    - Goals:
      ```
      clean install
      ```

14. **Save and run the job.**
    - Click **Build Now**.
    - ✅ Expected output:
      ```
      BUILD SUCCESS
      ```

15. **Repeat for `order-service`.**
    - Name it `order-service-CI`.

---

### **Part 4: Creating a Jenkins Pipeline**

16. **Install the Pipeline plugin if not installed.**
    - Go to **Manage Jenkins** → **Manage Plugins**.
    - Search for and install **Pipeline**.

17. **Create a pipeline job.**
    - Dashboard → **New Item** → Name: `Microservices-CI-Pipeline` → Select **Pipeline** → Click **OK**.

18. **Add the pipeline script.**
    - Under **Pipeline** → **Pipeline script**:
```groovy
pipeline {
    agent any
    stages {
        stage('Build user-service') {
            steps {
                script {
                    git branch:'main', url: '<user-service Git URL>'
                    dir('user-service') {
                        bat 'mvn clean install'
                    }
                }
            }
        }
        stage('Build order-service') {
            steps {
                script {
                    git branch:'main', url: '<order-service Git URL>'
                    dir('order-service') {
                        bat 'mvn clean install'
                    }
                }
            }
        }
    }
}
```

19. **Run the pipeline.**
    - Click **Build Now**.
    - ✅ Expected output:
      ```
      BUILD SUCCESS
      ```

---

### **Part 5: Deployment (Optional)**

20. **Configure webhook triggers.**
    - In your GitHub repository → **Settings** → **Webhooks** → Add URL to Jenkins webhook endpoint.

---

## **Conclusion**
You have:
- Installed Jenkins on **Windows**
- Created Freestyle and Pipeline jobs
- Automated build and optional deployment of two Spring Boot microservices

🎉 Your first CI pipeline is now live!
