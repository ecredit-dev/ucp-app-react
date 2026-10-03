pipeline { 
    agent any 
    tools { 
        nodejs 'Node_24' 
        sonarScanner 'SonarQubeScanner' // Configurado en Global Tools 
    } 
    environment { 
        SONAR_PROJECT_KEY = 'ucp-app-react' 
        SONAR_PROJECT_NAME = 'UCP React App' 
    } 
    stages { 
        // Etapa 1: Checkout (se mantiene igual) 
        stage('Checkout') { 
            steps { 
                git branch: 'main', url: 
'https://github.com/ecredit-dev/ucp-app-react.git' 
            } 
        } 
 
        // Nueva etapa: Análisis de SonarQube 
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
 
        // Etapa 2: Build (modificada para generar cobertura) 
        stage('Build') { 
            steps { 
                sh 'npm install' 
                sh 'npm run build' 
                sh 'npm run test:coverage' // Asegúrate de que tu package.json 
tenga este script 
            } 
        } 
 
        // Resto de las etapas se mantienen igual... 
        // ... 
    } 
    post { 
        always { 
            // Agregar notificación de calidad de SonarQube 
            script { 
                def qg = waitForQualityGate() 
                if (qg.status != 'OK') { 
                    error "Calidad no aprobada: ${qg.status}" 
                } 
            } 
      // Etapa 3: Ejecutar pruebas unitarias
       stage('Pruebas Unitarias') {
           steps {
               sh 'npm test -- --watchAll=false --ci --reporters=default --reporters=jest-junit' // Genera reporte JUnit
           }
           post {
               always {
                   junit 'junit.xml' // Publica reporte en Jenkins
                   archiveArtifacts artifacts: 'junit.xml', allowEmptyArchive: true
               }
           }
       }


   }


   // Post-actions (opcional)
   post {
       always {
           emailext (
               subject: "Pipeline ${currentBuild.result}: ucp-app-react #${env.BUILD_NUMBER}",
               body: """
                   Estado: ${currentBuild.result}
                   URL Build: ${env.BUILD_URL}
                   Detalles de Pruebas: ${env.BUILD_URL}testReport/
               """,
               to: 'dawian85@gmail.com' // Reemplaza con tu email
           )
       }
   }
