pipeline {
    agent any

    environment {
        // Nome do projeto no SonarQube (pode usar seu nome)
        SONAR_PROJECT_KEY = "mobead-enio-silva"
        SONAR_SCANNER = "sonarqube-scanner"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Fazendo checkout do repositório..."
                checkout scm
            }
        }

        stage('Build/Testes') {
            steps {
                echo "Executando build e testes..."
                sh '''
                    if [ -f package.json ]; then
                        npm install
                        npm run build || echo "Sem etapa de build"
                    else
                        echo "Nenhum arquivo package.json encontrado, prosseguindo..."
                    fi
                '''
            }
        }

        stage('Análise SonarQube') {
            steps {
                echo "Enviando análise para o SonarQube..."
                withSonarQubeEnv('sonarqube') {
                    sh """
                        ${env.SONAR_SCANNER} \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Aguardando validação do SonarQube..."
                timeout(time: 3, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Análise reprovada pelo SonarQube: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                echo "Realizando deploy no ambiente de DESENVOLVIMENTO..."
                sh '''
                    mkdir -p /var/www/mobead-dev
                    cp -r * /var/www/mobead-dev/
                '''
            }
        }

        stage('Aprovação para Produção') {
            steps {
                script {
                    def aprovado = input(
                        message: 'Liberar deploy em PRODUÇÃO?',
                        parameters: [booleanParam(name: 'Aprovar', defaultValue: false, description: 'Confirme para continuar')]
                    )
                    if (!aprovado) {
                        error "Deploy em produção não aprovado."
                    }
                }
            }
        }

        stage('Deploy PROD') {
            steps {
                echo "Realizando deploy no ambiente de PRODUÇÃO..."
                sh '''
                    mkdir -p /var/www/mobead-prod
                    cp -r * /var/www/mobead-prod/
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline finalizada com sucesso!"
        }
        failure {
            echo "❌ Falha na execução da pipeline. Verifique os logs."
        }
    }
}
