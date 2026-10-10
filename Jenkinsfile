pipeline { 
    agent any 
    tools { 
        nodejs 'Node_24' 
    }
    stages { 
        stage('Checkout') { 
            steps { 
                git branch: 'main', url: 'https://github.com/ecredit-dev/ucp-app-react.git' 
            }
        }
        stage('Build and Test') { 
            steps { 
                sh 'npm install' 
                sh 'npm run build' 
                sh 'npm run test:coverage' 
            } 
        }
 
        stage('Security Scan with Snyk') { 
            steps { 
                script { 
                    withCredentials([string(credentialsId: 'SNYK_API_TOKEN', variable: 'SNYK_TOKEN')]) { 
                        // Instalar CLI de Snyk 
                        sh 'npm install -g snyk' 
                        
                        // Autenticar 
                        sh 'snyk auth ${SNYK_TOKEN}' 
                        
                        // Ejecutar test de seguridad 
                        sh 'snyk test --all-projects --severity-threshold=high' 
                        
                        // Monitorear en Snyk (registra resultados en dashboard) 
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
                    sh 'ls -la build/ || echo "Directory build/ does not exist"' 
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
            publishHTML target: [ 
                allowMissing: false, 
                alwaysLinkToLastBuild: true, 
                keepAll: true, 
                reportDir: 'build', 
                reportFiles: 'index.html', 
                reportName: 'Demo Deploy' 
            ] 
            
            mail( 
                to: 'dawian85@gmail.com', 
                subject: "Build Status: ${currentBuild.currentResult}", 
                body: "Job: ${env.JOB_NAME}\\nEstado: ${currentBuild.currentResult}\\nURL: ${env.BUILD_URL}"
            ) 
            
            cleanWs() 
        } 
    } 
}
