pipeline {
    agent any

    environment {
        SONARQUBE = 'SonarQube-10'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/hasnahatti70/spring-boot-jenkins-ci-cd'
            }
        }

        stage('Build & Tests') {
            steps {
                sh 'mvn clean test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'mvn sonar:sonar -Dsonar.projectKey=springcrud'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Clean & Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage("Docker Build & Push") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub-Token', toolName: 'docker') {
                        def imageName = "spring-boot-prof-management"
                        def buildTag = "${imageName}:${BUILD_NUMBER}"
                        def latestTag = "${imageName}:latest"

                        sh "docker build -t ${imageName} -f Dockerfile.final ."
                        sh "docker tag ${imageName} abdeod/${buildTag}"
                        sh "docker tag ${imageName} abdeod/${latestTag}"
                        sh "docker push abdeod/${buildTag}"
                        sh "docker push abdeod/${latestTag}"
                        env.BUILD_TAG = buildTag
                    }
                }
            }
        }

        stage('Vulnerability scanning') {
            steps {
                sh "trivy image abdeod/${BUILD_TAG}"
            }
        }

        stage("Staging") {
            steps {
                sh 'docker-compose up -d'
            }
        }
    }
}
