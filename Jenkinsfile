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
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    app = docker.build("jmnaranjo09/train-schedule")
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('DeployToProduction') {
            when {
                branch 'master'
            }
            steps {
                input 'Deploy to Production?'
                milestone(1)
                script {
                    sh 'ssh -o StrictHostKeyChecking=no deployer@10.90.100.79 "docker pull jmnaranjo09/train-schedule:${env.BUILD_NUMBER}"'
                    try {
                        sh 'ssh -o StrictHostKeyChecking=no deployer@10.90.100.79 "docker stop train-schedule"'
                        sh 'ssh -o StrictHostKeyChecking=no deployer@10.90.100.79 "docker rm train-schedule"'
                    } catch (err) {
                        echo: 'caught error: $err'
                    }
                    sh 'ssh -o StrictHostKeyChecking=no deployer@10.90.100.79 "docker run --restart always --name train-schedule -p 8080:8080 -d jmnaranjo09/train-schedule:${env.BUILD_NUMBER}"'
                }
            }
        }
    }
}