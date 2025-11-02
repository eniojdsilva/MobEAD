pipeline {
    agent any

    environment {
        
        SONARQUBE_ENV = 'sonarqube'
        
        SONAR_SCANNER = 'sonarqube-scanner'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Realizando checkout do repositório Git..."
                checkout scm
            }
        }

        stage('Build/Testes') {
            steps {
                echo "Executando build e testes simulados..."
                sh 'echo "Build e testes concluídos com sucesso!"'
            }
        }

        stage('Análise SonarQube') {
            steps {
                echo "Iniciando análise de código com SonarQube..."
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    script {
                        def scannerHome = tool "${SONAR_SCANNER}"
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=mobead-enio-silva \
                                -Dsonar.projectName=mobead-enio-silva \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://192.168.1.15:9000
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "Aguardando resultado do Quality Gate..."
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                echo "Fazendo deploy em ambiente de desenvolvimento (DEV)..."
                sh 'echo "Deploy DEV concluído com sucesso!"'
            }
        }

        stage('Aprovação para Produção') {
            steps {
                echo "Aguardando aprovação para deploy em produção..."
                sh 'echo "Aprovação concluída com sucesso!"'
            }
        }

        stage('Deploy PROD') {
            steps {
                echo "Realizando deploy em ambiente de produção (PROD)..."
                sh 'echo "Deploy em produção concluído com sucesso!"'
            }
        }
    }

    post {
        success {
            echo "PIPELINE FINALIZADO COM SUCESSO! "
        }
        failure {
            echo "Ocorreu uma falha na pipeline."
        }
        always {
            echo "Status final da pipeline: ${currentBuild.currentResult}"
        }
    }
}
