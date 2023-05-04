pipeline {
    agent any
    tools {
        jdk 'OpenJDK-1.8.0' // specify the ID of the JDK to use
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running Build stage'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }

        stage('Deploy to staging') {
            when {
                branch 'master'
            }
            steps {
                echo 'Running Deploy Staging'
                sh '''
                sftp -o StrictHostKeyChecking=no deployer@10.90.100.76 << EOF
                cd /tmp/
                put dist/trainSchedule.zip
                quit EOF
                '''
                sh 'ssh -o StrictHostKeyChecking=no deployer@10.90.100.76 "sudo /usr/bin/systemctl stop train-schedule && sudo rm -rf /opt/train-schedule/* && sudo unzip -o /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule"'
            }
        }
        stage('Deploy to production') {
            when {
                branch 'master'
            }
            steps {
                echo 'Running Deploy Production'
                input('Does the staging server look good?')
                milestone(1)
                sh 'sftp -o StrictHostKeyChecking=no deployer@10.90.100.75 "cd /tmp/ && put dist/trainSchedule.zip && quit"'
                sh 'ssh -o StrictHostKeyChecking=no deployer@10.90.100.75 "sudo /usr/bin/systemctl stop train-schedule && sudo rm -rf /opt/train-schedule/* && sudo unzip -o /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule"'
            }
        }
    }
}
