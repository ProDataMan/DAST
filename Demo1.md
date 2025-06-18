## Demo 1: Steps

1. **Start ZAP with Desktop GUI**:
   - Open Git Bash and navigate to the ZAP directory:
     ```bash
     cd /d/D/GitRepos/DevOpsJunkies/DAST/ZAP_2_16_1
     ```
   - Start ZAP with the desktop GUI:
     ```bash
     java -jar zap-2.16.1.jar -host 0.0.0.0 -port 8080 -config api.disablekey=true -config api.enabled=true -config log4j.level=DEBUG
     ```
   - Wait 30–60 seconds for ZAP to initialize.
   - **Expected**: A ZAP desktop GUI window opens, showing tabs like “Quick Start,” “Sites,” “History,” and “Alerts” on the left or top, with the "Quick Start" tab active and displaying the "Automated Scan" section (as in your screenshot with "URL to attack" and "Attack" button).
   - **Troubleshooting**:
     - If the GUI doesn’t open, check Java version:
       ```bash
       java -version
       ```
       - Expect: `openjdk version "17.0.15"` or similar (11+ required).
     - Check logs for errors:
       ```bash
       cat D:/GitRepos/DevOpsJunkies/DAST/ZAP_2_16_1/zap.log | grep -i "error\|gui"
       ```
     - If errors persist, clear the ZAP config:
       ```bash
       rm -rf ~/.ZAP
       ```
       - Retry the start command.

2. **Verify Juice Shop is Running**:
   - Open a browser (e.g., Chrome) and navigate to `http://localhost:3000`.
     - **Expected**: The OWASP Juice Shop login page loads, showing a green “Juice Shop” logo.
   - In Git Bash, check the Docker container:
     ```bash
     docker ps
     ```
     - **Expected**: See `bkimminich/juice-shop` mapped to `0.0.0.0:3000->3000/tcp`.
   - If not running, start Juice Shop:
     ```bash
     docker run --rm -p 3000:3000 bkimminich/juice-shop
     ```
   - **Troubleshooting**:
     - If `http://localhost:3000` shows “Connection refused,” verify Docker Desktop is running and restart the container:
       ```bash
       docker stop $(docker ps -q -f ancestor=bkimminich/juice-shop)
       docker run --rm -p 3000:3000 bkimminich/juice-shop
       ```
     - Check port 3000:
       ```bash
       netstat -aon | grep :3000
       taskkill /PID <pid> /F
       ```

3. **Set the Target URL in ZAP GUI**:
   - In the ZAP desktop GUI, ensure the “Quick Start” tab is selected (it should be active by default, showing the "Automated Scan" section as in your screenshot).
   - In the "Automated Scan" section, locate the “URL to attack” field (a text box with `http://localhost:3000` already entered in your screenshot).
   - If the field is empty, click it and type `http://localhost:3000` (ensure “http,” not “https”).
   - Press Enter or click the “Check” button (if visible) to validate the URL.
     - **Expected**: The field retains `http://localhost:3000` with no error message.
   - **Troubleshooting**:
     - If an error says “URL not reachable,” verify Juice Shop in a browser (`http://localhost:3000`).
     - In Git Bash, test connectivity:
       ```bash
       curl -I http://localhost:3000
       ```
       - Expect: `HTTP/1.1 200 OK`.
     - If the “Quick Start” tab is missing, click “File” > “New Session” to reset the GUI, then recheck.

4. **Configure the Automated Scan**:
   - In the "Automated Scan" section of the Quick Start tab:
     - Check the box labeled “Use traditional spider” to crawl Juice Shop’s pages (e.g., login, search).
     - Check “Use ajax spider” if available (optional, for JavaScript-driven content like Juice Shop’s Angular frontend).
     - Find the “Scan Policy” dropdown (or “Attack Mode” setting) and select “Default Policy” or “Medium” strength.
       - **Note**: Medium balances speed and thoroughness, suitable for a 2–3 minute demo.
     - Uncheck any option labeled “Use ZAP as proxy” or “Configure browser proxy” to simplify the scan (no browser setup needed).
   - Confirm settings:
     - URL: `http://localhost:3000`
     - Traditional Spider: Enabled
     - AJAX Spider: Enabled (if checked)
     - Scan Policy: Default Policy/Medium
     - Proxy: Disabled
   - **Troubleshooting**:
     - If options are grayed out, ensure the URL is valid (`http://localhost:3000`).
     - If the scan policy dropdown is missing, click “Tools” > “Options” > “Active Scan” and set “Default Policy” manually.
     - If errors appear, check logs:
       ```bash
       cat D:/GitRepos/DevOpsJunkies/DAST/ZAP_2_16_1/zap.log
       ```

5. **Run the Automated Scan**:
   - In the "Automated Scan" section, click the “Attack” button (a green button or play icon as shown in your screenshot).
     - **Expected**: The “Active Scan” tab (bottom pane) shows a progress bar, and the “Alerts” tab (left or bottom) starts populating with findings.
   - Monitor progress for 2–3 minutes:
     - The “Sites” tab (left pane) lists URLs like `http://localhost:3000/#/search`.
     - The “Alerts” tab shows vulnerabilities (e.g., “Cross Site Scripting” with red flags for high severity).
   - **Note**: The traditional spider maps the site first (1–2 minutes), then the active scan tests for vulnerabilities (1–2 minutes).
   - **Troubleshooting**:
     - If the scan doesn’t start, verify the URL and settings, then click “Attack” again.
     - If no alerts appear after 3 minutes, extend the scan or select “High” strength in the Scan Policy.
     - Check logs for errors:
       ```bash
       cat D:/GitRepos/DevOpsJunkies/DAST/ZAP_2_16_1/zap.log | grep -i error
       ```
     - If the scan is slow, reduce scope:
       - In the “Sites” tab, right-click `http://localhost:3000` and select “Include in Context” > “Default Context.”
       - Restart the scan.

6. **Review the Scan Results**:
   - In the ZAP GUI, click the “Alerts” tab (left or bottom pane).
     - **Expected**: A list of vulnerabilities appears, with red flags for high-severity issues like “Cross Site Scripting (Reflected)” or “SQL Injection” (as indicated by “Attack complete - see the Alerts tab” in your screenshot).
   - Double-click an XSS alert to view details:
     - **URL**: e.g., `http://localhost:3000/#/search?q=<script>alert(1)</script>`
     - **Parameter**: e.g., `q`
     - **Evidence**: e.g., `<script>alert(1)</script>`
     - **Description**: Explains the issue (e.g., “Malicious script injection”).
     - **Solution**: Suggests remediation (e.g., “Sanitize user inputs with DOMPurify”).
   - Take a screenshot of an XSS alert:
     - Press `Win+Shift+S`, select the alert window, save as `demo1-xss.png` in `/d/D/GitRepos/DevOpsJunkies/DAST`.
   - **Troubleshooting**:
     - If no high-severity alerts, wait 5 minutes or rerun with “High” strength.
     - If the Alerts tab is empty, verify the “Sites” tab shows `http://localhost:3000` URLs.
     - Manually test an XSS vulnerability:
       - Open a browser and go to `http://localhost:3000/#/search?q=<script>alert(1)</script>`.
       - Expect a JavaScript alert popup (confirms the vulnerability).

7. **Showcase Automation Features and Generate Report**:
   - In the ZAP GUI, click “Tools” > “Options” > “Active Scan” to show automation settings:
     - Point out the “Run at startup” checkbox, explaining it enables scheduled scans.
     - **Highlight**: Scheduled scans integrate with CI/CD pipelines (foreshadowing Demo 2).
   - Generate an HTML report:
     - Click “Report” > “Generate HTML Report” in the top menu.
     - In the dialog, set the file name to `scan-report.html` and save to `/d/D/GitRepos/DevOpsJunkies/DAST`.
     - Click “Generate.”
     - **Expected**: A file `scan-report.html` is created.
   - Open `scan-report.html` in a browser (e.g., double-click in File Explorer).
     - **Expected**: A formatted report lists vulnerabilities, severity levels, and remediation steps.
   - **Troubleshooting**:
     - If the report option is missing, use the API as a fallback:
       ```bash
       curl -o /d/D/GitRepos/DevOpsJunkies/DAST/scan-report.html "http://localhost:8080/JSON/report/?formMethod=GET&reportType=html&fileName=scan-report.html"
       ```
     - If the report is empty, ensure the scan completed (Alerts tab has entries).
   - **Highlight**: Explain how HTML reports support DevSecOps by sharing results with developers and security teams.
