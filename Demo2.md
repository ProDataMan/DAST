## Demo 2: Steps

1. **Start Jenkins and Sample Web App**:
   - In Git Bash, stop any existing Jenkins container to ensure a fresh start:
     ```bash
     docker stop jenkins
     docker rm jenkins
     ```
   - Remove the existing `jenkins_home` volume to avoid reusing old configurations:
     ```bash
     docker volume rm jenkins_home
     ```
   - Start Jenkins on port 8085 with a new volume (for a fresh install):
     ```bash
     docker run -p 8085:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --name jenkins -d jenkins/jenkins:lts
     ```
   - Start DVWA (for a fresh install):
     ```bash
     docker run -p 8081:80 -d --name dvwa vulnerables/web-dvwa
     ```
   - Wait 2–3 minutes, then open a browser and go to `http://localhost:8085`.
     - **Expected**: Jenkins setup page; get the initial admin password:
       ```bash
       docker exec $(docker ps -q -f name=jenkins) cat /var/jenkins_home/secrets/initialAdminPassword
       ```
       - **Note**: Uses `docker ps -q` to dynamically get the latest `jenkins` container ID.
       - **Expected**: A 32-character password (e.g., `a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6`).
       - **Fallback**: If the file is missing, check logs:
         ```bash
         docker logs $(docker ps -q -f name=jenkins) 2>&1 | grep -i "password"
         ```
       - If still unavailable, recreate the container:
         ```bash
         docker stop $(docker ps -q -f name=jenkins)
         docker rm $(docker ps -q -f name=jenkins)
         docker volume rm jenkins_home
         docker run -p 8085:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --name jenkins -d jenkins/jenkins:lts
         ```
         - Wait 2–3 minutes, then retry:
           ```bash
           docker exec $(docker ps -q -f name=jenkins) cat /var/jenkins_home/secrets/initialAdminPassword
           ```
     - Enter the password in the “Unlock Jenkins” field, click “Continue,” install suggested plugins, and set an admin user (e.g., `admin`/`P@ssw0rd123`).
   - Verify DVWA at `http://localhost:8081` (setup with `admin`/`password`).
   - **Troubleshooting**: If ports conflict, ensure 8085 is free:
     ```bash
     netstat -aon | grep :8085
     taskkill /PID <pid> /F
     ```

2. **Create a Pipeline Job in Jenkins**:
   - In the Jenkins browser window, click “New Item” on the dashboard.
   - Enter “DAST-Pipeline” as the name, select “Pipeline,” and click “OK”.
   - In the “Pipeline” section, choose “Pipeline script” and paste:
     ```groovy
     pipeline {
         agent any
         stages {
             stage('Build') {
                 steps {
                     echo 'Building the application...'
                 }
             }
             stage('DAST Scan') {
                 steps {
                     sh 'curl -s -X POST http://localhost:8080/JSON/ascan/action/scan/?url=http://localhost:8081&recurse=true&inScopeOnly=true'
                     sh 'sleep 120'  # Wait for scan to start (adjust time as needed)
                 }
             }
             stage('Report') {
                 steps {
                     sh 'curl -s -o /var/jenkins_home/scan-report.html http://localhost:8080/OTHER/core/other/htmlreport/'
                     archiveArtifacts 'scan-report.html'
                 }
             }
         }
     }
     ```
   - Click “Save”.
   - **Troubleshooting**: If the script fails, ensure ZAP API is enabled and accessible.

3. **Start ZAP Locally on Port 8080**:
   - In Git Bash, navigate to the ZAP directory and start ZAP locally:
     ```bash
     cd /d/D/GitRepos/DevOpsJunkies/DAST/ZAP_2_16_1
     java -jar zap-2.16.1.jar -host 0.0.0.0 -port 8080 -config api.disablekey=true -config api.enabled=true -config log4j.level=DEBUG
     ```
   - Wait 30–60 seconds for ZAP to initialize.
   - **Expected**: A ZAP desktop GUI window opens, showing tabs like “Quick Start,” “Sites,” and “Alerts”.
   - **Troubleshooting**:
     - If the GUI doesn’t open, check logs:
       ```bash
       cat D:/GitRepos/DevOpsJunkies/DAST/ZAP_2_16_1/zap.log | grep -i "error\|gui"
       ```
     - Ensure port 8080 is free (Jenkins is on 8085):
       ```bash
       netstat -aon | grep :8080
       taskkill /PID <pid> /F
       ```

4. **Trigger the Pipeline**:
   - In Jenkins, open the “DAST-Pipeline” job and click “Build Now”.
   - Click the build number (e.g., #1) under “Build History” to view progress.
   - **Expected**: Console output shows “Building the application...” and API calls executing.
   - **Troubleshooting**: If the build fails, ensure ZAP is running on port 8080 and API is enabled.

5. **Monitor Pipeline Execution**:
   - In the build’s “Console Output” tab, watch for:
     - `Building the application...`
     - API scan initiation and report generation.
   - Wait 2–3 minutes for the scan to complete (adjust `sleep` time if needed).
   - **Troubleshooting**: If stuck, check ZAP logs:
     ```bash
     cat D:/GitRepos/DevOpsJunkies/DAST/ZAP_2_16_1/zap.log | grep -i error
     ```

6. **Display Results in Jenkins Dashboard**:
   - After the build completes, return to the “DAST-Pipeline” job page.
   - Click “Back to Project” and find `scan-report.html` under “Build Artifacts”.
   - Click to view in the browser.
   - **Expected**: Report shows vulnerabilities (e.g., XSS) with severity levels.
   - **Highlight**: Explain how pipeline integration enables frequent, automated scans.
   - **Troubleshooting**: If no artifacts, ensure the API call succeeded; rerun the build.
