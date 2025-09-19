# **Lab 14: Set Up a CI Pipeline with Jenkins for Automated Testing and Deployment of Microservices (Spring Boot 3.4.5)**

## **Objective**
Learn how to install and configure **Jenkins** to automate the build, testing, and deployment process for **Spring Boot 3.4.5** microservices. You will create a CI pipeline that builds two microservices (`user-service` and `order-service`) and optionally deploys them.

---

## **Lab Steps**

### **Part 1: Installing Jenkins (Windows)**

1) **Install Java (Jenkins prerequisite).**  
   - Check Java:
     ```cmd
     java -version
     ```
     ✅ Expected output:
     ```
     openjdk version "17.x.x"
     ```
   - If not found, install **JDK 17** from https://adoptium.net

2) **Download Jenkins.**  
   - Get the Windows installer from https://www.jenkins.io/download/

3) **Install Jenkins.**  
   - Run the installer and choose **Run Jenkins as a Service**.

4) **Start Jenkins.**  
   - Open **Services** (Start Menu) and start the **Jenkins** service.

5) **Verify Jenkins is running.**  
   - Open a browser and go to:
     ```
     http://localhost:8080
     ```

6) **Unlock Jenkins.**  
   - Retrieve the initial admin password:
     ```cmd
     type C:\Jenkins\secrets\initialAdminPassword
     ```
   - Paste it into the Jenkins setup screen.

7) **Install suggested plugins.**  
   - Choose **Install suggested plugins**.

8) **Create an admin user.**  
   - Complete the setup with an admin account.

> **What is Jenkins?**  
> Jenkins is an open‑source automation server. In this lab, you’ll use it to automatically build and test your Spring Boot microservices when you push code.

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

**Open in IntelliJ and update the `pom.xml` for the Producer (`order-service`).**  
- Open **IntelliJ IDEA** → **File → Open…** → select `C:\studentCloudLabs\lab14\order-service`.  
- Wait for Maven import to finish (lower‑right status bar).  
- Add the following dependency to `order-service/pom.xml`:
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>
</dependency>
```

**Open in IntelliJ and update the `pom.xml` for the Producer (`user-service`).**  
- Open **File → Open…** → select `C:\studentCloudLabs\lab14\user-service`.  
- Wait for Maven import to finish.  
- Add the following dependency to `user-service/pom.xml`:
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>
</dependency>
```

> 💡 Tip: In IntelliJ, open **Terminal** via **View → Tool Windows → Terminal** (or **Alt+F12**) inside each project to run build/test and Git commands.

---

## **Git/GitHub Setup — Recommended Quick Path (Option A)**
*Most students already authenticated Git in earlier labs. Use the simple Git commands below in the **IntelliJ Terminal** opened inside each service folder.*

### ✅ Option A — Simple Git Workflow (IntelliJ Terminal)

> In IntelliJ’s **Project** tool window, **right‑click** the folder (`user-service` or `order-service`) → **Open in Terminal**.

#### 1) Initialize and commit (first time only)
**user-service**
```powershell
git init; git add .; git commit -m "Initial commit – user-service"
```

**order-service**
```powershell
git init; git add .; git commit -m "Initial commit – order-service"
```

#### 2) Connect to a remote and push
Pick one path per service:

- **If you already created an empty GitHub repo on the website:**
  **user-service**
  ```powershell
  git remote add origin https://github.com/<your-username>/user-service.git; git branch -M main; git push -u origin main
  ```
  **order-service**
  ```powershell
  git remote add origin https://github.com/<your-username>/order-service.git; git branch -M main; git push -u origin main
  ```

- **If the remote is already set (you cloned earlier):**
  ```powershell
  git remote -v
  ```
  Then push:
  ```powershell
  git push -u origin main
  ```

#### 3) Ongoing updates
Run inside the changed service folder:
```powershell
git add .; git commit -m "Describe your change"; git push
```

**Troubleshooting**
- Check identity: `git config --global user.name` and `git config --global user.email`
- If `origin` exists but wrong URL:
```powershell
git remote set-url origin https://github.com/<your-username>/<repo-name>.git
```

> 🔍 Prefer GUI? You can also use IntelliJ’s VCS actions: **VCS → Enable Version Control Integration… (Git)**, **Commit** (**Ctrl+K**), then **Push** (**Ctrl+Shift+K**).

---

## **(Optional) Option B — GitHub CLI (gh)**
Use this if you haven’t authenticated GitHub before and prefer CLI automation. **Optional** (run in the IntelliJ Terminal or PowerShell).

**Install & login (PowerShell as Administrator)**
```powershell
winget install --id GitHub.cli
gh --version
gh auth login
```
Follow prompts: **GitHub.com** → **HTTPS** → browser sign‑in → **Authorize**.

**Create & push (run inside each folder)**

`user-service`
```powershell
gh repo create user-service --public --add-readme --confirm; git add .; git commit -m "Initial commit – user-service"; git branch -M main; git push -u origin main
```

`order-service`
```powershell
gh repo create order-service --public --add-readme --confirm; git add .; git commit -m "Initial commit – order-service"; git branch -M main; git push -u origin main
```

---

### **Verify Maven Surefire Plugin**
Ensure both services include (or are managed to include) the Surefire plugin so tests run in CI:
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
> 💡 Spring Boot manages most dependency versions—avoid pinning versions unnecessarily.

---

## **Part 3: Configure Jenkins (Freestyle Jobs)**

1) **Create job for `user-service`.**  
   - Jenkins dashboard → **New Item** → Name: `user-service-CI` → **Freestyle project** → **OK**.

2) **Set Git repository.**  
   - **Source Code Management** → **Git** → Repository URL: *(your `user-service` repo URL)*  
   - **Branch Specifier**: `*/main`

3) **Add Maven build step.**  
   - **Build** → **Invoke top-level Maven targets** → Goals:
     ```
     clean install
     ```

4) **Build.**  
   - Click **Build Now** → expect `BUILD SUCCESS`.

5) **Repeat for `order-service`** with job name `order-service-CI`.

---

## **Part 4: Jenkins Pipeline (Build Both Services)**

1) **Install Pipeline plugin (if missing).**  
   - **Manage Jenkins** → **Manage Plugins** → install **Pipeline**.

2) **Create a pipeline job.**  
   - **New Item** → Name: `Microservices-CI-Pipeline` → **Pipeline** → **OK**.

3) **Pipeline script** (replace URLs with your repos):
```groovy
pipeline {
    agent any
    stages {
        stage('Build user-service') {
            steps {
                script {
                    git branch: 'main', url: '<user-service Git URL>'
                    dir('user-service') {
                        bat 'mvn clean install'
                    }
                }
            }
        }
        stage('Build order-service') {
            steps {
                script {
                    git branch: 'main', url: '<order-service Git URL>'
                    dir('order-service') {
                        bat 'mvn clean install'
                    }
                }
            }
        }
    }
}
```

4) **Run the pipeline.**  
   - Click **Build Now** → expect `BUILD SUCCESS`.

---

## **Part 5: (Optional) Webhook Triggers**
In each GitHub repo: **Settings** → **Webhooks** → add your Jenkins endpoint to trigger builds on push.

---

## **Conclusion**
You have:
- Installed Jenkins on **Windows**
- Pushed both microservices to GitHub (simple Git or optional GitHub CLI via IntelliJ Terminal)
- Created Freestyle and Pipeline jobs
- Automated builds and optional deployment hooks

🎉 Your first CI pipeline is now live!
