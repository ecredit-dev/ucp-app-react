pipeline { 
  agent any 
  tools { 
    nodejs 'Node_24' 
} 
environment { 
    SONAR_PROJECT_KEY = 'ucp-app-react' 
    SONAR_PROJECT_NAME = 'UCP React App' 
    SONAR_SCANNER_HOME = tool 'SonarQubeScanner' 
    SONAR_HOST_URL = 'http://sonarqube:9000' 
} 
stages { 
    stage('Checkout') { 
        steps { 
            git branch: 'main', url: 
'https://github.com/ecredit-dev/ucp-app-react.git' 
  } 
} 
stage('Build and Test') { 
        steps { 
               sh 'npm install' 
               sh 'npm run build' 
               sh 'npm run test:coverage' 
           } 
       } 
 
      stage('SonarQube Analysis') {
    steps {
        script {
            withSonarQubeEnv('SonarQube') {
                sh """
                    \${SONAR_SCANNER_HOME}/bin/sonar-scanner \\
                      -Dsonar.projectKey=\${SONAR_PROJECT_KEY} \\
                      -Dsonar.projectName='\${SONAR_PROJECT_NAME}' \\
                      -Dsonar.sources=src \\
                      -Dsonar.login=\${SONAR_AUTH_TOKEN} \\
                      -Dsonar.javascript.node=\$(tool 'Node_24')/bin/node \\
                      -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
                """
            }

            def qg = waitForQualityGate()
            if (qg.status != 'OK') {
                error "Quality Gate failed: ${qg.status}"
            }
        }
    }
}
                   
 
       stage('Security Scan with Snyk') { 
           steps { 
               script { 
                   withCredentials([string(credentialsId: 'SNYK_API_TOKEN', 
variable: 'SNYK_TOKEN')]) { 
                       // Instalar CLI de Snyk 
                       sh 'npm install -g snyk' 
                       
                       // Autenticar 
                       sh 'snyk auth ${SNYK_TOKEN}' 
                       
                       // Ejecutar test de seguridad 
                       sh 'snyk test --all-projects --severity-threshold=high' 
                       
                       // Opcional: Monitorear en Snyk (registra resultados en 
//dashboard) 
                       sh 'snyk monitor --all-projects' 
                   } 
               } 
           } 
       } 
 
       stage('Parallel Tests') { 
           parallel { 
               stage('Chrome Tests') { 
                   steps { 
                       script { 
                           try { 
                               sh 'npm test -- --browser=chrome --watchAll=false --ci --reporters=jest-junit' 
                               junit 'junit-chrome.xml' 
                           } catch (err) { 
                               echo "Chrome tests failed: ${err}" 
                               currentBuild.result = 'UNSTABLE' 
                           } 
                       } 
                   } 
               } 
               stage('Firefox Tests') { 
                   steps { 
                       script { 
                           try { 
                               sh 'npm test -- --browser=firefox --watchAll=false --ci --reporters=jest-junit' 
                               junit 'junit-firefox.xml' 
                           } catch (err) { 
                               echo "Firefox tests failed: ${err}" 
                               currentBuild.result = 'UNSTABLE' 
                           } 
                       } 
                   } 
               } 
           } 
       } 
 
       stage('Deploy (Simulated)') { 
           steps { 
               script { 
                   // Verifica que el directorio build existe 
                   sh 'ls -la build/ || echo "Directory build/ does not exist"' 
           
                   // Crea prod y copia con verificación 
                   sh ''' 
                       mkdir -p prod 
                       echo "Contents of build/:" 
                       ls -la build/ 
                       echo "Copying files..." 
                       cp -r build/* prod/ || echo "Copy failed" 
                       echo "Contents of prod/:" 
                       ls -la prod/ 
                   ''' 
               } 
           } 
       } 
   } 
   post { 
       always { 
 
           // Publicar HTML 
           publishHTML target: [ 
               allowMissing: false, 
               alwaysLinkToLastBuild: true, 
               keepAll: true, 
               reportDir: 'build', 
               reportFiles: 'index.html', 
               reportName: 'Demo Deploy' 
           ] 
           
           // Notificación por email 
           mail( 
               to: 'dawian85@gmail.com', 
               subject: "Build Status: ${currentBuild.currentResult}", 
               body: "Job: ${env.JOB_NAME}\nEstado: 
          ${currentBuild.currentResult}\nURL: ${env.BUILD_URL}" 
           ) 
           
           // Limpiar workspace (esto puede quedarse fuera del node) 
           cleanWs() 
       } 
   } 
} 
