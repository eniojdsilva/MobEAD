pipeline {
    agent any

    environment {
        // Nome da imagem que será construída localmente
        imageName = "mobead-prod"
        // Caminho remoto para o deploy
        remoteUser = "devlab"
        remoteHost = "192.168.1.9"
        remotePath = "/home/devlab/deploys/mobead-prod"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "🔄 Realizando checkout do código..."
                checkout scm
            }
        }

        stage('Build/Testes') {
            steps {
                echo "🧪 Etapa de build e testes (se houver testes locais)"
                sh 'echo "Nenhum teste configurado ainda..."'
            }
        }

        stage('Análise SonarQube') {
            environment {
                scannerHome = tool 'sonarqube-scanner' // mesmo nome configurado no Jenkins
            }
            steps {
                echo "🔍 Executando análise no SonarQube..."
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=mobead-enio-silva \
                        -Dsonar.projectName=mobead-enio-silva \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=admin \
                        -Dsonar.password=admin
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    echo "🚦 Aguardando resultado do Quality Gate..."
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "🐳 Construindo imagem Docker local..."
                    sh "docker build -t ${imageName}:latest ."
                }
            }
        }

        stage('Deploy PROD (Servidor DEVLAB)') {
            steps {
                script {
                    echo "🚀 Enviando aplicação para o servidor de produção (${remoteHost})..."

                    // Remove container antigo e substitui pela nova versão
                    sh """
                        ssh ${remoteUser}@${remoteHost} '
                            docker rm -f ${imageName} 2>/dev/null || true &&
                            docker rmi ${imageName}:latest 2>/dev/null || true &&
                            cd ${remotePath}/mobead-enio-silva-ci-cd &&
                            docker build -t ${imageName}:latest . &&
                            docker run -d --name ${imageName} -p 8080:80 ${imageName}:latest
                        '
                    """
                }
            }
        }

        stage('Cleanup Local') {
            steps {
                echo "🧹 Limpando imagens locais não utilizadas..."
                sh 'docker image prune -f || true'
            }
        }
    }

    post {
        success {
            echo "Pipeline concluído com sucesso!"
        }
        failure {
            echo "A pipeline falhou. Verifique os logs acima."
        }
    }
}
