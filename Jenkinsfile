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
                    sftp -o StrictHostKeyChecking=no deployer@10.90.100.76 << EOC
                    cd /tmp/
                    put dist/trainSchedule.zip
                    quit
                    EOC
                '''
                sh 'ssh -o StrictHostKeyChecking=no deployer@10.90.100.76 "/usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && /usr/bin/systemctl start train-schedule"'
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
                sh '''
                    sftp -o StrictHostKeyChecking=no deployer@10.90.100.75 << EOC
                    cd /tmp/
                    put dist/trainSchedule.zip
                    quit
                    EOC
                    ssh -o StrictHostKeyChecking=no deployer@10.90.100.75 << EOC
                    sudo /usr/bin/systemctl stop train-schedule
                    sudo rm -rf /opt/train-schedule/*
                    sudo unzip /tmp/trainSchedule.zip -d /opt/train-schedule
                    sudo /usr/bin/systemctl start train-schedule
                    EOC
                  '''
            }
        }
    }
}
