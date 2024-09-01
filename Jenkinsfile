pipeline {
    agent any
    environment {
        SONAR_HOME = tool "sonar"
    }
    stages {
        stage("Code Clone from GitHub") {
            steps {
                git url: "https://github.com/Sarvagya82/two-tier-flask-app-master.git", branch: "jenkins"
            }
        }
        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=two-tier-flask-app -Dsonar.projectKey=two-tier-flask-app"
                }
            }
        }
        stage("OWASP Dependency Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWAS_DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage("Build & Test") {
            steps {
                sh "docker build . -t flaskapp"
            }
        }
        stage("Push to DockerHub") {
            steps {
                withCredentials([usernamePassword(credentialsId: "dockerHub", passwordVariable: "dockerHubPass", usernameVariable: "dockerHubUser")]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker tag flaskapp ${env.dockerHubUser}/flaskapp:latest"
                    sh "docker push ${env.dockerHubUser}/flaskapp:latest"
                }
            }
        }
        stage("Deploy") {
            steps {
                sh "docker-compose down && docker-compose up -d"
            }
        }
    }
}
