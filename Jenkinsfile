pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
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
                    app = docker.build("ifathi7/train-schedule")
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
                        "docker pull ifathi7/train-schedule:${env.BUILD_NUMBER}\""
                        try {
                            "docker stop train-schedule\"
                            "docker rm train-schedule\"
                        } catch (err) {
                            echo: 'caught error: $err'
                        }
                        "docker run --restart always --name train-schedule -p 8080:8080 -d ifathi7/train-schedule:${env.BUILD_NUMBER}\"
                    }                
            }
        }
    }
}
