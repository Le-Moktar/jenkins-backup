pipeline {
    agent any
    environment {
        BRANCH_NAME = 'main'
        GIT_URL = 'https://github.com/Le-Moktar/jenkins-backup.git'
        TEMP_DIR = "/tmp/jenkins_backup_tmp"
        SOURCE_DIR = "/var/lib/jenkins"
        BACKUP_DIR = "/tmp"
        BACKUP_FILE = "jenkins_backup_${new Date().format('yyyyMMdd')}.tar.gz"
        S3_BUCKET = "my-wm-s3-backup-bucket"
    }
    stages {
        stage('Git checkout') {
            steps {
                git branch: "${BRANCH_NAME}", url: "${GIT_URL}"
            }
        }
        stage('Prepare Backup Directory') {
            steps {
                script {
                    echo "Creating a temporary backup directory..."
                    sh """
                    mkdir -p ${TEMP_DIR}
                    rsync -a ${SOURCE_DIR}/ ${TEMP_DIR}/
                    """
                }
            }
        }
        stage('Create Backup') {
            steps {
                script {
                    echo "Compressing temporary backup directory..."
                    sh """
                    tar --warning=no-file-changed -czf ${BACKUP_DIR}/${BACKUP_FILE} -C ${TEMP_DIR} .
                    """
                }
            }
        }
        
        stage('Upload to S3') {
            steps {
                script {
                    echo "Uploading backup to S3..."
                    sh """
                    aws s3 cp ${BACKUP_DIR}/${BACKUP_FILE} s3://${S3_BUCKET}/
                    """
                }
            }
        }
        stage('Clean Up') {
            steps {
                script {
                    echo "Cleaning up local backup file..."
                    sh """
                    rm -rf ${TEMP_DIR}
                    rm -f ${BACKUP_DIR}/${BACKUP_FILE}
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Backup and upload completed successfully."
        }
        failure {
            echo "Pipeline failed. Please check the logs for errors."
        }
    }
}
