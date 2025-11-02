pipeline {
    agent any

    environment {
        SONAR_SCANNER = 'sonarqube-scanner'
        SONAR_PROJECT_KEY = 'mobead-enio-silva'
        SONAR_URL = 'http://192.168.1.15:9000'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build/Testes') {
            steps {
                echo 'Executando build/testes (placeholder)...'
                sh 'echo "OK"'
            }
        }

        stage('Análise SonarQube') {
            steps {
                // usa o servidor que você já cadastrou em:
                // Manage Jenkins > Configure System > SonarQube Servers
                withSonarQubeEnv('SonarQube') {
                    script {
                        def scannerHome = tool "${SONAR_SCANNER}"
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                              -Dsonar.projectName=${SONAR_PROJECT_KEY} \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=${SONAR_URL}
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                // igual aparecia na linha da #6: "paused for 4s"
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                // na #6 isso aqui falava rápido e por isso as próximas ficaram vermelhas
                echo 'Fazendo deploy em DEV (aqui entrava o ssh antigo)...'
                sh 'exit 1'   // mantém o comportamento de falha que você viu na #6
            }
        }

        stage('Aprovação para Produção') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                input message: 'Publicar em PRODUÇÃO?', ok: 'Sim'
            }
        }

        stage('Deploy PROD') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                echo 'Deploy em PRODUÇÃO'
            }
        }
    }

    post {
        always {
            echo "Pipeline finalizado com status: ${currentBuild.currentResult}"
        }
    }
}
