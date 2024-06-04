Problem Statement 1:
Title: Containerisation and Deployment of Wisecow Application on Kubernetes
Project Repository: https://github.com/nyrahul/wisecow Wisecow App
Objective: To containerize and deploy the Wisecow application, hosted in the
above-mentioned GitHub repository, on a Kubernetes environment with secure TLS
communication.
Requirements:
Dockerization:
● Develop a Dockerfile for creating a container image of the Wisecow
application.
Kubernetes Deployment:
● Craft Kubernetes manifest files for deploying the Wisecow application
in a Kubernetes environment.
● The Wisecow app must be exposed as a Kubernetes service for
accessibility.
Continuous Integration and Deployment (CI/CD):
● Implement a GitHub Actions workflow for:
● Automating the build and push of the Docker image to a
container registry whenever changes are committed to the
repository.
● Continuous Deployment [Challenge Goal]: Automatically
deploy the updated application to the Kubernetes environment
following successful image builds.
TLS Implementation [Challenge Goal]:
● Ensure that the Wisecow application supports secure TLS
communication.
Expected Artifacts:
● A private GitHub repository containing:
● The Wisecow application source code.
● The Dockerfile for the application.

● Kubernetes manifest files for deployment.
● The CI/CD pipeline configuration.
● A GitHub Actions workflow file for facilitating Continuous Build
and Deployment (CI/CD)

Access Control:
● The GitHub repository should be set to public.

End Goal: The successful containerisation and deployment of the Wisecow
application to the Kubernetes environment with an automated CI/CD pipeline and
secured with TLS communication.


Sol : To successfully complete the containerization and deployment of the Wisecow application on Kubernetes with secure TLS communication, you'll need to perform several steps. Here is a comprehensive guide to achieve the outlined objectives:
1. Dockerization
Create a Dockerfile for the Wisecow application:
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME Wisecow

# Run app.py when the container launches
CMD ["python", "app.py"]

2.  Kubernetes Deployment
Create Kubernetes manifest files:

Deployment manifest (deployment.yaml):

apiVersion: apps/v1
kind: Deployment
metadata:
  name: wisecow-deployment
  labels:
    app: wisecow
spec:
  replicas: 3
  selector:
    matchLabels:
      app: wisecow
  template:
    metadata:
      labels:
        app: wisecow
    spec:
      containers:
      - name: wisecow
        image: <your-dockerhub-username>/wisecow:latest
        ports:
        - containerPort: 80
        
Service manifest (service.yaml):

apiVersion: v1
kind: Service
metadata:
  name: wisecow-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: wisecow
    
Ingress manifest (ingress.yaml):
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wisecow-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  rules:
  - host: wisecow.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: wisecow-service
            port:
              number: 80
  tls:
  - hosts:
    - wisecow.yourdomain.com
    secretName: wisecow-tls

Cert-Manager Cluster Issuer manifest (cluster-issuer.yaml):

apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: your-email@example.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - http01:
        ingress:
          class: nginx
          
3. Continuous Integration and Deployment (CI/CD)
Create a GitHub Actions workflow file (.github/workflows/ci-cd.yml):

name: CI/CD Pipeline

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v1

    - name: Cache Docker layers
      uses: actions/cache@v2
      with:
        path: /tmp/.buildx-cache
        key: ${{ runner.os }}-buildx-${{ github.sha }}
        restore-keys: |
          ${{ runner.os }}-buildx-

    - name: Build and push Docker image
      uses: docker/build-push-action@v2
      with:
        push: true
        tags: <your-dockerhub-username>/wisecow:latest
        cache-from: type=local,src=/tmp/.buildx-cache
        cache-to: type=local,dest=/tmp/.buildx-cache

    - name: Set up Kubernetes
      uses: azure/setup-kubectl@v1
      with:
        version: v1.19.6

    - name: Deploy to Kubernetes
      env:
        KUBECONFIG: ${{ secrets.KUBECONFIG }}
      run: |
        kubectl apply -f k8s/deployment.yaml
        kubectl apply -f k8s/service.yaml
        kubectl apply -f k8s/ingress.yaml
        
4. TLS Implementation
Ensure the Wisecow application supports secure TLS communication using the Cert-Manager and Let's Encrypt configuration provided above. The ingress.yaml manifest includes the necessary configurations to handle TLS termination.

5. Access Control
Set your GitHub repository to public and include the necessary files:

Source Code: Ensure the Wisecow application source code is in the repository.
Dockerfile: Place the Dockerfile in the root of the repository.
Kubernetes Manifests: Place the Kubernetes manifest files (deployment.yaml, service.yaml, ingress.yaml, cluster-issuer.yaml) in a directory, e.g., k8s/.
GitHub Actions Workflow: Place the workflow file in .github/workflows/ci-cd.yml.
By following these steps, you will have successfully containerized the Wisecow application, deployed it on Kubernetes with secure TLS communication, and set up a CI/CD pipeline using GitHub Actions.


Problem Statement 2:
Please choose any two objectives from the list below and attempt to achieve them
using either Bash or Python.

1. System Health Monitoring Script:
Develop a script that monitors the health of a Linux system. It should check
CPU usage, memory usage, disk space, and running processes. If any of
these metrics exceed predefined thresholds (e.g., CPU usage > 80%), the
script should send an alert to the console or a log file.
2. Automated Backup Solution:
Write a script to automate the backup of a specified directory to a remote
server or a cloud storage solution. The script should provide a report on the
success or failure of the backup operation.
3. Log File Analyzer:
Create a script that analyzes web server logs (e.g., Apache, Nginx) for
common patterns such as the number of 404 errors, the most requested
pages, or IP addresses with the most requests. The script should output a
summarized report.
4. Application Health Checker:

Please write a script that can check the uptime of an application and
determine if it is functioning correctly or not. The script must accurately
assess the application's status by checking HTTP status codes. It should be
able to detect if the application is 'up', meaning it is functioning correctly, or
'down', indicating that it is unavailable or not responding.


Sol:  2. Automated Backup Solution:
Write a script to automate the backup of a specified directory to a remote
server or a cloud storage solution. The script should provide a report on the
success or failure of the backup operation.

Certainly! Below is a Python script that automates the backup of a specified directory to a remote server using rsync. The script will generate a report on the success or failure of the backup operation and log it.

To use this script, you need to have rsync installed on your system and SSH access to the remote server.

import os
import subprocess
import logging
from datetime import datetime

# Configuration
LOCAL_DIR = '/path/to/local/directory'
REMOTE_DIR = 'user@remote_server:/path/to/remote/directory'
LOG_FILE = 'backup.log'

# Configure logging
logging.basicConfig(filename=LOG_FILE, level=logging.INFO, format='%(asctime)s - %(message)s')

def run_backup(local_dir, remote_dir):
    try:
        # Run rsync command
        result = subprocess.run(
            ['rsync', '-avz', local_dir, remote_dir],
            check=True,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE
        )
        logging.info("Backup completed successfully")
        return True, result.stdout.decode('utf-8')
    except subprocess.CalledProcessError as e:
        logging.error("Backup failed with error: %s", e.stderr.decode('utf-8'))
        return False, e.stderr.decode('utf-8')

def main():
    logging.info("Starting backup process")
    success, output = run_backup(LOCAL_DIR, REMOTE_DIR)
    
    # Print report to console
    if success:
        print("Backup completed successfully.")
        print(output)
    else:
        print("Backup failed.")
        print(output)
    
    logging.info("Backup process finished")

if __name__ == "__main__":
    main()

Explanation:
Configuration:

LOCAL_DIR: Path to the local directory you want to back up.
REMOTE_DIR: Path to the remote directory in the format user@remote_server:/path/to/remote/directory.
LOG_FILE: File where the log will be saved.
Logging Configuration:

Logs are written to backup.log file with timestamps for easy monitoring.
Function run_backup:

Uses subprocess.run to execute the rsync command for backing up files.
Logs the success or failure of the backup operation.
Main Function:

Starts the backup process and logs the start time.
Calls run_backup to perform the backup.
Prints a success or failure report to the console.
Logs the completion of the backup process.

How to Run the Script:

Save the script to a file, for example, backup_script.py.
Ensure rsync is installed on your system. You can install it using the following command:
bash
sudo apt-get install rsync  # for Debian/Ubuntu
sudo yum install rsync      # for CentOS/RHEL

Make sure you have SSH access to the remote server configured.

Run the script:
bash
python backup_script.py

This script will perform the backup using rsync, log the results to backup.log, and print the outcome to the console. Adjust the directory paths and logging settings as needed.

3. Log File Analyzer:
Create a script that analyzes web server logs (e.g., Apache, Nginx) for
common patterns such as the number of 404 errors, the most requested
pages, or IP addresses with the most requests. The script should output a
summarized report.

Log File Analyzer:

Create a script that analyzes web server logs (e.g., Apache, Nginx) for common patterns such as the number of 404 errors, the most requested pages, or IP addresses with the most requests. The script should output a summarized report.
Sol:
Objective 3: Log File Analyzer

To achieve this objective, I'll write a Python script that analyzes web server logs for common patterns such as the number of 404 errors, the most requested pages, and IP addresses with the most requests. The script will output a summarized report.
 Code:
 import re
from collections import Counter

def analyze_log_file(log_file_path):
    # Open and read the log file
    with open(log_file_path, 'r') as file:
        log_data = file.readlines()

    # Initialize counters
    status_codes = Counter()
    requested_pages = Counter()
    ip_addresses = Counter()

    # Regular expressions for common log file formats
    ip_regex = r'\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}'
    status_code_regex = r'\" (\d{3}) '
    requested_page_regex = r'\"GET (.+?) HTTP'

    # Analyze each log entry
    for line in log_data:
        # Extract IP address, status code, and requested page
        ip = re.search(ip_regex, line).group()
        status_code = re.search(status_code_regex, line).group(1)
        requested_page = re.search(requested_page_regex, line).group(1)

        # Update counters
        status_codes[status_code] += 1
        requested_pages[requested_page] += 1
        ip_addresses[ip] += 1

    # Print summarized report
    print("Summary Report:")
    print(f"Total Requests: {len(log_data)}")
    print("Status Codes:")
    for code, count in status_codes.items():
        print(f"  {code}: {count}")
    print("Most Requested Pages:")
    for page, count in requested_pages.most_common(5):
        print(f"  {page}: {count}")
    print("IP Addresses with Most Requests:")
    for ip, count in ip_addresses.most_common(5):
        print(f"  {ip}: {count}")

if __name__ == "__main__":
    # Specify path to the web server log file
    log_file_path = "/path/to/access.log"
    
    # Call analyze_log_file function
    analyze_log_file(log_file_path)
This script reads a web server log file, extracts relevant information such as IP addresses, status codes, and requested pages using regular expressions, and then generates a summarized report. It counts the occurrences of each status code, requested page, and IP address, and outputs the top 5 entries for requested pages and IP addresses with the most requests.

To use this script:

Replace "/path/to/access.log" with the actual path to your web server log file.
Save the script to a file (e.g., log_file_analyzer.py).
Run the script using python log_file_analyzer.py.
You can customize the script further to include additional analysis based on your specific log file format or requirements. Additionally, you can extend the script to handle log rotation, analyze different log formats, or output the report to a file for further analysis.








