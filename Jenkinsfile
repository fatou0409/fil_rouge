pipeline {
    agent any

    environment {
        DOCKER_USER      = 'fatou0409'
        BACKEND_IMAGE    = "${DOCKER_USER}/profilapp-backend"
        FRONTEND_IMAGE   = "${DOCKER_USER}/profilapp-frontend"
        MIGRATE_IMAGE    = "${DOCKER_USER}/profilapp-migrate"
        SONARQUBE_URL    = "http://localhost:9000"
        SONARQUBE_TOKEN  = credentials('fafa')
    }

    stages {
        stage('Cloner le dépôt') {
            steps {
                git branch: 'main', url: 'https://github.com/fatou0409/projet_fil_rouge.git'
            }
        }

        stage('Build des images') {
            steps {
                bat 'docker build -t %BACKEND_IMAGE%:latest ./Backend-main/odc'
                bat 'docker build -t %FRONTEND_IMAGE%:latest ./Frontend-main'
                bat 'docker build -t %MIGRATE_IMAGE%:latest ./Backend-main/odc'
            }
        }

        stage('Analyse SonarQube - Backend') {
            steps {
                dir('Backend-main/odc') {
                    withEnv(["SONAR_TOKEN=${SONARQUBE_TOKEN}"]) {
                        bat '''
                            "C:\\Users\\hp\\Desktop\\SonarScanner\\sonar-scanner\\bin\\sonar-scanner.bat" ^
                              -Dsonar.projectKey=backend ^
                              -Dsonar.sources=. ^
                              -Dsonar.host.url=%SONARQUBE_URL% ^
                              -Dsonar.login=%SONAR_TOKEN%
                        '''
                    }
                }
            }
        }

        stage('Analyse SonarQube - Frontend') {
            steps {
                dir('Frontend-main') {
                    withEnv(["SONAR_TOKEN=${SONARQUBE_TOKEN}"]) {
                        bat '''
                            "C:\\Users\\hp\\Desktop\\SonarScanner\\sonar-scanner\\bin\\sonar-scanner.bat" ^
                              -Dsonar.projectKey=frontend ^
                              -Dsonar.sources=. ^
                              -Dsonar.host.url=%SONARQUBE_URL% ^
                              -Dsonar.login=%SONAR_TOKEN%
                        '''
                    }
                }
            }
        }

        stage('Push des images sur Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: 'jenk', url: '']) {
                    bat 'docker push %BACKEND_IMAGE%:latest'
                    bat 'docker push %FRONTEND_IMAGE%:latest'
                    bat 'docker push %MIGRATE_IMAGE%:latest'
                }
            }
        }

        stage('Terraform - Déploiement sur Kubernetes') {
            steps {
                // Récupère le kubeconfig stocké comme secret fichier
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    // Expose-le dans la variable KUBECONFIG pour Terraform
                    withEnv(["KUBECONFIG=%KUBECONFIG_FILE%"]) {
                        dir('terraform') {
                            // Vérifie que Terraform est disponible
                            bat 'terraform -version'
                            // Initialise et applique la configuration
                            bat 'terraform init'
                            bat 'terraform plan -out=tfplan'
                            bat 'terraform apply -auto-approve tfplan'
                        }
                    }
                }
            }
        }
    }
}
