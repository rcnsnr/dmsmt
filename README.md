> Maintenance status: archived / historical project. This repository is kept as a reference for older Linux operations automation work.

# Disk Monitoring and System Maintenance Tool

This project is a comprehensive **disk monitoring and system maintenance tool** designed to track disk utilization, identify large files, manage log rotation, and detect zombie processes. It integrates seamlessly with Jenkins pipelines for scheduled checks and generates alerts via Microsoft Teams and Slack. The tool is designed to work across various environments, including:

- **Oracle Enterprise Linux**  
- **Oracle DB, MongoDB, Cassandra, Redis, Kafka**  
- **OpenShift clusters**  
- **Jenkins, Bitbucket, Jira, Confluence servers**  
- **Application servers running Java and other test environments**  

---

## Project Structure

```bash
/.
│
├── modules/
│   ├── disk_usage.py           # Disk monitoring and cleanup
│   ├── process_monitor.py      # Zombie process detection and termination
│   ├── alerting.py             # Teams and Slack notifications
│   ├── system_detection.py     # Filesystem and application detection
│   └── elk_logger.py           # ELK Logging integration
│
├── Dockerfile                  # Docker containerization
├── main.py                     # Main script integrating all modules
├── Jenkinsfile                 # Jenkins pipeline for automation
└── requirements.txt            # Python dependencies

```

## Features

### 1. Disk Usage Monitoring

- **Threshold-based Monitoring:**  
  - Monitors disk usage and sends alerts if a specified threshold is exceeded.  
  - Supports multiple mount points and paths.  
- **Large File Detection:**  
  - Scans for files exceeding a specified size and archives or deletes them.  
- **Log Rotation and Archiving:**  
  - Automatically compresses or deletes log files older than a specified number of days.  

### 2. File Type-Specific Actions

- **Log Files (`.log`):** Compress, delete, or rotate based on retention days.  
- **Backup Files (`.bak, .tar, .gz`):** Archive or delete after a certain period.  
- **Temporary Files (`.tmp, .swp`):** Immediate cleanup of temporary files.  
- **Database Dumps (`.sql, .db`):** Archive or move older dumps.  
- **Media Files (`.mp4, .mkv`):** Move or archive large media files.  
- **ISO Images (`.iso, .img`):** Archive or delete after a period.  

### 3. Zombie Process Detection and Cleanup

- **Zombie Process Detection:**  
  - Scans for zombie processes (`ps aux | grep 'Z'`) and identifies their parent processes (PPID).  
- **Parent Process Cleanup:**  
  - Automatically terminates parent processes (`kill -9`) to eliminate zombie processes.  
- **Threshold-Based Alerts:**  
  - Sends Teams or Slack notifications when the number of zombie processes exceeds a defined threshold.  

### 4. Jenkins Integration

- **Pipeline Integration:**  
  - Designed to run as part of Jenkins pipelines.  
  - Supports parameterized builds for flexible disk checks and process monitoring.  

### 5. ELK Integration for Logging

- **Centralized Logging:**  
  - Sends disk monitoring and process cleanup logs to ELK (Elasticsearch, Logstash, Kibana).  
  - Provides better visualization and monitoring.  

---

### 6. Application and Filesystem-Specific Actions

- **Oracle DB (on ext4 or XFS):**

  - Cleans up Oracle database logs and archive files automatically.  
  - Detects database dump files and moves them to archival storage.  

- **MongoDB, Cassandra, Redis:**

  - Scans `/var/lib` directories for large `.wt` or `.sst` files.  
  - Performs automatic compaction or backup on large files.  

---

### 7. Logging and Monitoring

- **Local Logging:**  

  - All actions (disk usage alerts, file deletions, zombie process detection) are logged in `/var/log/disk_monitor.log`.  
  - Supports INFO, WARNING, and ERROR log levels.  

- **Jenkins Build Artifacts:**  

  - Disk usage reports and logs are attached as artifacts to Jenkins builds.  
  - Failed builds trigger automatic alerts to Microsoft Teams or Slack.  

---

## Technology Stack

- **Language:** Python 3.9+  
- **Dependencies:**  
  - `psutil` – Disk and process monitoring  
  - `requests` – Microsoft Teams and Slack integration  
  - `subprocess` – Command execution for disk and process management  
  - `python-dotenv` – Environment variable management  
  - `elasticsearch` – ELK integration  
- **Tools:**  
  - Jenkins (for scheduled execution)  
  - Microsoft Teams / Slack (for alerting)  
  - Docker (for containerization)  

---

## Installation

### Requirements

- Python 3.9+  
- Jenkins (optional for CI/CD integration)  
- Microsoft Teams or Slack Webhook URL for notifications  
- ELK stack for centralized logging (optional)  

### Setup

1. Clone the repository

2. Install dependencies:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

3. Set up Microsoft Teams Webhook URL in `.env` file:

```ini
TEAMS_WEBHOOK_URL=https://outlook.office.com/webhook/YOUR-WEBHOOK-URL
ELK_HOST=your-elk-host
ELK_PORT=9200
```

---

### Usage

#### Run Manually

```bash
python3 main.py --threshold 85 --log_retention_days 30 --scan_path /var/log --check_zombies true
```

##### Run via Jenkins Pipeline

```groovy
pipeline {
    agent any

    environment {
        PYTHON_VERSION = '3.9'
    }

    parameters {
        string(name: 'THRESHOLD', defaultValue: '85', description: 'Disk usage threshold')
        string(name: 'LOG_RETENTION_DAYS', defaultValue: '30', description: 'Log retention period (days)')
        string(name: 'SCAN_PATH', defaultValue: '/', description: 'Directory to scan')
        booleanParam(name: 'CHECK_ZOMBIES', defaultValue: true, description: 'Enable zombie process detection')
    }

    stages {
        stage('Disk and Process Monitoring') {
            steps {
                sh '''
                source venv/bin/activate
                python main.py --threshold ${THRESHOLD} --log_retention_days ${LOG_RETENTION_DAYS} --scan_path ${SCAN_PATH} --check_zombies ${CHECK_ZOMBIES}
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
```
