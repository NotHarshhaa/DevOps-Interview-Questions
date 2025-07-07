# Automation & Scripting Interview Questions

## **Beginner-Level (1-20) Questions**  

### **1. What is automation in DevOps?**  

Automation in DevOps refers to scripting repetitive tasks like provisioning, configuration, deployment, and monitoring to improve efficiency and reduce errors.  

### **2. What are the benefits of scripting in DevOps?**  

- Reduces manual effort  
- Increases consistency and repeatability  
- Improves efficiency and speed  
- Reduces errors and enhances security  

### **3. What is Bash scripting?**  

Bash scripting is writing command-line instructions in a script file (`.sh`) to automate tasks in Unix/Linux environments.  

### **4. How do you write a basic Bash script?**  

```bash
#!/bin/bash
echo "Hello, DevOps!"
```

Save the file (`script.sh`), make it executable (`chmod +x script.sh`), and run it (`./script.sh`).  

### **5. What is the difference between Bash and Shell scripting?**  

Bash is a type of shell, but shell scripting can also be done in other shells like **sh, csh, and zsh**. Bash provides more advanced scripting features.  

### **6. What are variables in Bash?**  

Variables store values and are defined without a `$` sign but accessed using `$`.  

```bash
name="DevOps"
echo "Hello, $name"
```

### **7. What is Python scripting used for in DevOps?**  

- Infrastructure as Code (IaC)  
- CI/CD automation  
- Log analysis  
- Cloud automation (AWS, Azure, GCP SDKs)  

### **8. How do you define a function in Python?**  

```python
def greet():
    print("Hello, DevOps!")
greet()
```

### **9. What is YAML, and where is it used?**  

YAML (Yet Another Markup Language) is a human-readable format used for **Kubernetes configurations, Ansible playbooks, CI/CD pipelines**, etc.  

### **10. What is a YAML file example?**  

```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "80:80"
```

### **11. How do you write a simple Groovy script?**  

```groovy
println "Hello, DevOps!"
```

Groovy is used in **Jenkins pipelines and automation tasks**.  

### **12. What is the shebang (`#!`) in a script?**  

The shebang (`#!/bin/bash` or `#!/usr/bin/python3`) specifies the interpreter for executing the script.  

### **13. What are loops in Bash?**  

Bash supports `for`, `while`, and `until` loops. Example:  

```bash
for i in {1..5}; do echo "Iteration $i"; done
```

### **14. What are conditional statements in Bash?**  

`if-else` statements execute different code based on conditions.  

```bash
if [ $USER == "root" ]; then echo "Admin access"; else echo "User access"; fi
```

### **15. How do you read input in Bash?**  

```bash
echo "Enter name: "
read name
echo "Hello, $name"
```

### **16. How do you create a Python virtual environment?**  

```bash
python3 -m venv myenv
source myenv/bin/activate
```

### **17. How do you parse JSON in Python?**  

```python
import json
data = '{"name": "DevOps"}'
parsed = json.loads(data)
print(parsed["name"])
```

### **18. What is the `awk` command in Bash?**  

`awk` is used for text processing. Example:  

```bash
awk '{print $1}' file.txt
```

Extracts the first column from `file.txt`.  

### **19. How do you comment in YAML?**  

Use `#` for comments.  

```yaml
# This is a comment
name: DevOps
```

### **20. How do you declare variables in Groovy?**  

```groovy
def name = "DevOps"
println name
```

---

## **Intermediate-Level (21-40) Questions**  

### **21. How do you pass arguments to a Bash script?**  

```bash
#!/bin/bash
echo "Hello, $1!"
```

Run: `./script.sh DevOps` → Output: `Hello, DevOps!`  

### **22. How do you handle errors in Bash scripts?**  

Use `set -e` to stop execution on errors.  

### **23. What is an Ansible playbook?**  

A YAML file that defines automation tasks for servers.  

### **24. How do you handle exceptions in Python?**  

```python
try:
    print(1 / 0)
except ZeroDivisionError:
    print("Cannot divide by zero")
```

### **25. How do you schedule a script with Cron?**  

Edit `crontab -e` and add:  

```bash
0 5 * * * /path/to/script.sh
```

Runs the script daily at 5 AM.  

### **26. How do you execute a Groovy script in Jenkins?**  

Use `script {}` inside a Jenkins pipeline.  

### **27. How do you create a list in Python?**  

```python
mylist = [1, 2, 3]
print(mylist[0])
```

### **28. What is `sed` in Bash?**  

Used for text replacement. Example:  

```bash
sed -i 's/old/new/g' file.txt
```

### **29. How do you define a dictionary in Python?**  

```python
mydict = {"name": "DevOps"}
print(mydict["name"])
```

### **30. How do you validate a YAML file?**  

Use `yamllint` or `kubectl apply -f --dry-run=client`.  

### **31. How do you install Python modules?**  

```bash
pip install requests
```

### **32. What is Jenkins pipeline syntax for automation?**  

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo "Building..."
            }
        }
    }
}
```

### **33. How do you iterate over a dictionary in Python?**  

```python
for key, value in mydict.items():
    print(key, value)
```

### **34. How do you set environment variables in Bash?**  

```bash
export VAR="DevOps"
```

### **35. What is an Ansible role?**  

A reusable way to organize Ansible tasks, handlers, and templates.  

### **36. What is a multiline string in YAML?**  

```yaml
message: |
  Line 1
  Line 2
```

### **37. What is an associative array in Bash?**  

```bash
declare -A myarray
myarray[name]="DevOps"
echo ${myarray[name]}
```

### **38. How do you run a shell command in Python?**  

```python
import os
os.system("ls")
```

### **39. What is `jq` in Linux?**  

Used to parse JSON. Example:  

```bash
cat data.json | jq '.name'
```

### **40. How do you exit a script with a status code?**  

```bash
exit 1
```

---

## **Advanced-Level (41-60) Questions**  

### **41. How do you debug a Bash script?**  

Use `set -x` for debugging:  

```bash
#!/bin/bash
set -x
echo "Debugging mode enabled"
```

### **42. How do you trap signals in a Bash script?**  

```bash
trap "echo 'Script interrupted'; exit" SIGINT SIGTERM
```

Catches `Ctrl+C` (SIGINT) and terminates gracefully.  

### **43. What is the difference between `$(command)` and backticks `` `command` `` in Bash?**  

Both execute commands, but `$(command)` is preferred as it's **nestable**.  

### **44. How do you handle multiline commands in a Bash script?**  

Use `\` for line continuation:  

```bash
echo "This is a \
multiline command"
```

### **45. How do you use conditionals inside a YAML file?**  

With Jinja2 templating in Ansible:  

```yaml
tasks:
  - name: Install package
    yum:
      name: httpd
    when: ansible_os_family == "RedHat"
```

### **46. How do you encrypt secrets in YAML?**  

Use **Ansible Vault**:  

```bash
ansible-vault encrypt secrets.yaml
```

### **47. How do you execute a Python script inside a Bash script?**  

```bash
python3 <<EOF
print("Hello from Python")
EOF
```

### **48. What is the difference between `continue` and `break` in Bash loops?**  

- `break` exits the loop entirely.  
- `continue` skips the current iteration.  

### **49. How do you parse a JSON file in Bash?**  

Use `jq`:  

```bash
cat data.json | jq '.key'
```

### **50. How do you use arrays in Groovy?**  

```groovy
def arr = ["a", "b", "c"]
println arr[0]
```

### **51. What is a Jenkins shared library in Groovy?**  

A reusable **pipeline function** stored in Git.  

### **52. How do you set a timeout for a script in Bash?**  

```bash
timeout 10s ./script.sh
```

### **53. How do you execute a Bash function in a subshell?**  

```bash
(my_function)
```

Runs in a new shell, not affecting the parent script.  

### **54. How do you use Python to send an HTTP request?**  

```python
import requests
response = requests.get("https://example.com")
print(response.text)
```

### **55. How do you handle authentication in a Python script?**  

```python
import requests
requests.get("https://example.com", auth=("user", "pass"))
```

### **56. How do you find and replace text in YAML files?**  

Use `yq`:  

```bash
yq e '.name="NewValue"' -i config.yaml
```

### **57. How do you define a multi-stage Jenkins pipeline in Groovy?**  

```groovy
pipeline {
    agent any
    stages {
        stage('Build') { steps { echo "Building..." } }
        stage('Test') { steps { echo "Testing..." } }
    }
}
```

### **58. What is an inline script in a CI/CD pipeline?**  

Running a script **directly inside a pipeline** (e.g., GitHub Actions, Jenkins).  
Example in GitHub Actions:  

```yaml
jobs:
  build:
    steps:
      - run: echo "Inline script execution"
```

### **59. How do you execute a script remotely via SSH in Bash?**  

```bash
ssh user@server 'bash -s' < local_script.sh
```

### **60. How do you ensure idempotency in an Ansible playbook?**  

Ansible modules ensure **idempotency** by only making changes when needed.  


### **61. How do you ensure idempotency in a Bash script used for automation?**

**Answer:**  
Idempotency means running the script multiple times produces the same result without causing unintended side effects. To ensure idempotency in Bash scripts, you should:

- Check the current state before making changes (e.g., verify if a package is installed before installing).
- Use conditional statements to skip steps if already done.
- Avoid destructive commands without checks.
- Use flags or lock files to prevent concurrent runs.

Example:

```bash
if ! dpkg -l | grep -q "nginx"; then
  apt-get install -y nginx
fi
```

This prevents reinstalling nginx if it’s already installed.

### **62. Explain how you would debug a complex Bash script that is failing intermittently.**

**Answer:**  
To debug a complex Bash script:

- Use `set -x` at the start to enable execution tracing and see each command as it runs.
- Use `set -e` to exit immediately on errors.
- Insert `echo` statements or logging to track variable values and flow.
- Check for race conditions or environment dependencies causing intermittent failures.
- Use `trap` to catch signals and errors and log them.
- Run the script in a controlled environment to isolate external factors.

### **63. What are the differences between declarative and scripted Jenkins pipelines? When would you use each?**

**Answer:**  
- **Declarative Pipeline:**  
  - Uses a more structured and simpler syntax with predefined blocks (`pipeline`, `stages`, `steps`).  
  - Easier to read and maintain, designed for most CI/CD workflows.  
  - Supports built-in error handling and post actions.

- **Scripted Pipeline:**  
  - Uses Groovy scripting language, more flexible and powerful.  
  - Allows complex logic, loops, and conditionals not easily done declaratively.  
  - Requires deeper Groovy knowledge.

**Use cases:**  
- Use declarative for standard CI/CD pipelines with straightforward stages.  
- Use scripted when you need advanced logic, dynamic stages, or complex workflows.

### **64. How do you handle secrets management in YAML files for Ansible playbooks?**

**Answer:**  
Secrets should never be stored in plain YAML files. Best practices include:

- Use **Ansible Vault** to encrypt sensitive variables and files.  
- Store secrets in encrypted files and decrypt them at runtime.  
- Use environment variables or external secret managers (HashiCorp Vault, AWS Secrets Manager) and inject secrets dynamically.  
- Avoid hardcoding secrets in playbooks or version control.

Example command to create an encrypted file:

```bash
ansible-vault create secrets.yml
```

### **65. How do you parse JSON data in a Bash script?**

**Answer:**  
Bash does not natively parse JSON, so you use tools like `jq`:

```bash
json='{"name":"devops","age":5}'
name=$(echo $json | jq -r '.name')
echo $name  # Output: devops
```

`jq` allows querying and extracting JSON fields easily.

### **66. How can you trap signals in a Bash script and why is it important?**

**Answer:**  
Use the `trap` command to catch signals like `SIGINT` (Ctrl+C) or `SIGTERM` to perform cleanup or graceful shutdown:

```bash
trap 'echo "Script interrupted"; exit 1' SIGINT SIGTERM
```

This is important to:

- Clean up temporary files or resources.  
- Prevent partial or corrupted state.  
- Log interruptions for debugging.

### **67. Describe how you would create a multi-stage Jenkins pipeline for a microservices application.**

**Answer:**  
A multi-stage Jenkins pipeline for microservices typically includes:

- **Build stage:** Compile and build each microservice container image.  
- **Test stage:** Run unit tests and integration tests per microservice.  
- **Publish stage:** Push container images to a registry.  
- **Deploy stage:** Deploy microservices to Kubernetes or other environments, possibly with Helm charts.  
- **Approval stage:** Manual or automated approval before production deployment.  
- **Production deploy stage:** Deploy to production with blue/green or canary strategies.

This is implemented in Jenkinsfile with `stages` and parallel execution for microservices.

### **68. What is the difference between `$(command)` and backticks `` `command` `` in Bash? Which one is preferred and why?**

**Answer:**  
- Both execute a command and substitute its output.  
- `$(command)` is preferred because it is more readable, can be nested easily, and avoids confusion with backticks inside strings.  
- Backticks are older syntax and harder to read especially when nested.

Example:

```bash
result=$(ls -l)
```

### **69. How do you ensure idempotency and error handling in Ansible roles?**

**Answer:**  
- Use **`when`** conditions to check states before making changes.  
- Use **`changed_when`** and **`failed_when`** to control task outcomes.  
- Use **handlers** to trigger actions only when changes occur.  
- Use **`ignore_errors`** cautiously with proper logging.  
- Test roles extensively in different environments.

### **70. How do you create and use a Python virtual environment in a CI/CD pipeline?**

**Answer:**  
- Create a virtual environment:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

- Use the virtual environment to isolate dependencies and avoid conflicts.  
- In CI/CD, activate the venv before running tests or deployment scripts to ensure consistent environment.

These questions and answers cover advanced scripting, automation, CI/CD pipelines, configuration management, and best practices, providing a strong challenge for DevOps engineer interviews related to your list. If you want, I can provide more questions on specific topics like Kubernetes, Docker, or monitoring.


---

## **📢 Contribute & Stay Updated**  

💡 **Want to contribute?**  
We **welcome contributions!** If you have insights, new tools, or improvements, feel free to submit a **pull request**.  

📌 **How to Contribute?**

- Read the **[CONTRIBUTING.md](https://github.com/NotHarshhaa/DevOps-Interview-Questions/blob/master/CONTRIBUTING.md)** guide.  
- Fix errors, add missing topics, or suggest improvements.  
- Submit a **pull request** with your updates.  

📢 **Stay Updated:**  
⭐ **Star the repository** to get notified about new updates and additions.  
💬 **Join discussions** in **[GitHub Issues](https://github.com/NotHarshhaa/DevOps-Interview-Questions/issues)** to suggest improvements.  

---

## **🌍 Community & Support**  

🔗 **GitHub:** [@NotHarshhaa](https://github.com/NotHarshhaa)  
📝 **Blog:** [ProDevOpsGuy](https://blog.prodevopsguy.xyz)  
💬 **Telegram Community:** [Join Here](https://t.me/prodevopsguy)  

![Follow Me](https://imgur.com/2j7GSPs.png)
