pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'sonarqube'
        SONAR_PROJECT_KEY = 'mobead-enio-silva'
        DOCKER_IMAGE_DEV = 'mobead-dev'
        DOCKER_IMAGE_PROD = 'mobead-prod'
        PROD_SERVER = '192.168.1.9'
        PROD_USER = 'devlab'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📦 Fazendo checkout do código...'
                checkout scm
            }
        }

        stage('Build/Testes') {
            steps {
                echo '⚙️ Executando build e testes locais...'
                sh '''
                    if [ -f package.json ]; then
                        npm install
                        npm test || true
                    else
                        echo "Nenhum package.json encontrado — ignorando testes..."
                    fi
                '''
            }
        }

        stage('Análise SonarQube') {
            steps {
                echo '🔍 Enviando análise para o SonarQube...'
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh """
                        /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/sonarqube-scanner/sonar-scanner/bin/sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    echo '⏳ Aguardando resultado do Quality Gate...'
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy DEV (Local)') {
            steps {
                echo '🚀 Realizando deploy no ambiente DEV...'
                sh '''
                    docker stop mobead-dev || true && docker rm mobead-dev || true
                    docker build -t mobead-dev .
                    docker run -d -p 8080:8080 --name mobead-dev mobead-dev
                '''
            }
        }

        stage('Aprovação para Produção') {
            steps {
                script {
                    def userInput = input(
                        id: 'Proceed1', message: 'Deseja prosseguir com o deploy em PRODUÇÃO?',
                        parameters: [
                            choice(name: 'Confirmação', choices: 'NÃO\nSIM', description: 'Confirmar deploy em produção')
                        ]
                    )
                    if (userInput != 'SIM') {
                        error('🚫 Deploy em produção cancelado pelo usuário.')
                    }
                }
            }
        }

        stage('Deploy PROD (Remoto 192.168.1.9)') {
            steps {
                echo '🚀 Realizando deploy remoto em PRODUÇÃO (192.168.1.9)...'
                sh '''
                    ssh ${PROD_USER}@${PROD_SERVER} "docker stop ${DOCKER_IMAGE_PROD} || true && docker rm ${DOCKER_IMAGE_PROD} || true"
                    scp -r /var/lib/jenkins/workspace/mobead-enio-silva-ci-cd/* ${PROD_USER}@${PROD_SERVER}:/home/${PROD_USER}/deploys/mobead-prod/
                    ssh ${PROD_USER}@${PROD_SERVER} "cd /home/${PROD_USER}/deploys/mobead-prod && docker build -t ${DOCKER_IMAGE_PROD} ."
                    ssh ${PROD_USER}@${PROD_SERVER} "docker run -d -p 8080:8080 --name ${DOCKER_IMAGE_PROD} ${DOCKER_IMAGE_PROD}"
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline finalizada com sucesso!'
        }
        failure {
            echo '❌ Falha na pipeline. Verifique os logs para detalhes.'
        }
    }
}
