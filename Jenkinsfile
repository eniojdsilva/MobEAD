pipeline {
    agent any

    environment {
        // Configurações globais
        SONARQUBE_ENV = 'sonarqube' // nome configurado no Jenkins (Gerenciar Jenkins → Configurações do SonarQube)
        SONAR_PROJECT_KEY = 'mobead-enio-silva'
        SONAR_HOST_URL = 'http://192.168.1.15:9000'
        DEPLOY_USER = 'devlab'
        DEPLOY_HOST = '192.168.1.9'
        DEPLOY_PATH = '/home/devlab/deploys/mobead-prod'
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📦 Fazendo checkout do repositório...'
                checkout scm
            }
        }

        stage('Build/Testes') {
            steps {
                echo '🔧 Projeto HTML estático - sem build necessário.'
            }
        }

        stage('Análise SonarQube') {
            steps {
                echo '🔍 Enviando análise para o SonarQube...'
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh '''
                        ${SONAR_SCANNER} \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy DEV (local)') {
            steps {
                echo "🚧 Testando build local do Docker..."
                sh '''
                    docker build -t mobead-dev .
                    docker stop mobead-dev || true
                    docker rm mobead-dev || true
                    docker run -d -p 8081:80 --name mobead-dev mobead-dev
                '''
            }
        }

        stage('Aprovação para Produção') {
            steps {
                input message: "🚀 Deseja liberar o deploy em PRODUÇÃO (192.168.1.9)?", ok: "Sim, liberar"
            }
        }

        stage('Deploy PROD (Servidor 192.168.1.9)') {
            steps {
                echo "🚀 Iniciando deploy em PRODUÇÃO (192.168.1.9)..."
                sh '''
                    # Cria pasta de deploy se não existir
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} "mkdir -p ${DEPLOY_PATH}"

                    # Copia arquivos do Jenkins pro servidor remoto
                    scp -r * ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/

                    # Build da imagem Docker no servidor remoto
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} "
                        cd ${DEPLOY_PATH} && \
                        docker build -t mobead-prod .
                    "

                    # Para e remove o container antigo
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} "
                        docker stop mobead-prod || true && \
                        docker rm mobead-prod || true
                    "

                    # Sobe o novo container
                    ssh ${DEPLOY_USER}@${DEPLOY_HOST} "
                        docker run -d -p 8080:80 --name mobead-prod mobead-prod
                    "

                    echo "Deploy finalizado com sucesso no servidor ${DEPLOY_HOST}"
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executado com sucesso!'
        }
        failure {
            echo 'Falha detectada na pipeline. Verifique os logs.'
        }
    }
}
