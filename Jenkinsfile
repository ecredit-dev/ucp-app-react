pipeline {
    agent any

    tools {
        nodejs 'Node_24' // Nombre definido en Global Tool Configuration
        sonarScanner 'SonarQubeScanner' // Configurado en Global Tools 
    }

    environment { 
        SONAR_PROJECT_KEY = 'ucp-app-react' 
        SONAR_PROJECT_NAME = 'UCP React App' 
    } 

    stages {
        // Etapa 1: Checkout del código desde GitHub
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ecredit-dev/ucp-app-react.git'
            }
        }

        // Etapa 2: Instalar dependencias y build del proyecto
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
                sh 'npm run test:coverage'
            }
        }
stage('Security Scan with Snyk') { 
    steps { 
        script { 
            withCredentials([string(credentialsId: 'SNYK_API_TOKEN', variable: 
'SNYK_TOKEN')]) { 
                try { 
                    // 1. Instalar Snyk CLI 
                    sh 'npm install -g snyk' 
                     
                    // 2. Autenticación 
                    sh 'snyk auth ${SNYK_TOKEN}' 
                     
                    // 3. Test de vulnerabilidades (fail-on si hay 
//vulnerabilidades altas/críticas) 
                    sh 'snyk test --all-projects --severity-threshold=high' 
                     
                    // 4. Monitoreo continuo (opcional) 
                    sh 'snyk monitor --all-projects' 
                     
                    // 5. Generar reporte HTML (opcional) 
                    sh 'snyk test --all-projects --json-file-output=snyk_results.json' 
                    sh ''' 
                        npm install -g snyk-to-html 
                        snyk-to-html -i snyk_results.json -o snyk_report.html 
                    ''' 
                     
                    // Publicar reporte Snyk 
                    publishHTML target: [ 
                        allowMissing: true, 
                        alwaysLinkToLastBuild: true, 
                        keepAll: true, 
                        reportDir: '.', 
                        reportFiles: 'snyk_report.html', 
                        reportName: 'Snyk Security Report' 
                    ] 
                     
                } catch (err) { 
                    echo "Snyk scan failed: ${err}" 
                    // Marcar build como inestable si hay vulnerabilidades 
                    currentBuild.result = 'UNSTABLE' 
                } 
            } 
        } 
} 
}

        
        // Etapa 3: Análisis de SonarQube
        stage('SonarQube Analysis') { 
            steps { 
                withSonarQubeEnv('SonarQube') { 
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.projectName=${SONAR_PROJECT_NAME} \
                        -Dsonar.sources=src \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONAR_AUTH_TOKEN} \
                        -Dsonar.javascript.node=${NODEJS_HOME}/bin/node \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                    '''
                } 
            } 
        }

        // Etapa 4: Quality Gate (Espera la respuesta de SonarQube)
        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Quality Gate falló: ${qg.status}"
                        }
                    }
                }
            }
        }

        // Etapa 5: Ejecutar pruebas unitarias
        stage('Pruebas Unitarias') {
            steps {
                sh 'npm test -- --watchAll=false --ci --reporters=default --reporters=jest-junit'
            }
            post {
                always {
                    junit 'junit.xml'
                    archiveArtifacts artifacts: 'junit.xml', allowEmptyArchive: true
                }
            }
        }
    }

    // Bloque Post Unificado
    post {
        always {
            emailext (
                subject: "Pipeline ${currentBuild.result}: ucp-app-react #${env.BUILD_NUMBER}",
                body: """
                    Estado: ${currentBuild.result}
                    URL Build: ${env.BUILD_URL}
                    Detalles de Pruebas: ${env.BUILD_URL}testReport/
                """,
                to: 'dawian85@gmail.com'
            )
        }
    }
}
