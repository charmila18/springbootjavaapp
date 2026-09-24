pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        ACR_SERVER       = 'democontainerregistry12.azurecr.io'
        IMAGE_NAME       = 'springbootdemo'
        IMAGE_TAG        = 'latest'
        DEPLOYMENT_NAME  = 'petclinic'
        K8S_NAMESPACE    = 'default'
        EMAIL_FROM       = 'charmila535@gmail.com'
        EMAIL_RECIPIENTS = 'kammelacharmila@gmail.com'
    }

    stages {
        stage('Checkout from Git')
        {
            steps {
                git branch: 'main' ,url: 'https://github.com/charmila18/springbootjavaapp.git'
            }
        }
        stage ('Validate with Maven')
        {
            steps{
                sh 'mvn validate'
            }
        }
        stage('Compile with Maven')
        {
            steps{
                sh 'mvn compile'
            }
        }
        stage('Test with Maven')
        {
            steps{
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis')
        {
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    mvn sonar:sonar \
                        -Dsonar.organization=springboot123 \
                        -Dsonar.projectKey=springbootapp \
                        -Dsonar.projectName=springbootapp \
                        -Dsonar.java.binaries=target/classes
                        '''
                }
                
            }
        }
         stage('Package with Maven')
        {
            steps{
                sh 'mvn package'
            }
        }
        stage('Docker Build') {
            steps {
                 sh 'docker build -t $ACR_SERVER/$IMAGE_NAME:$IMAGE_TAG .'
                  }
               
            }
        stage('Trivy Image Scan') {
            steps {
                sh '''
                    trivy image --severity HIGH,CRITICAL --format table \
                        -o trivy-image-report.txt \
                        $ACR_SERVER/$IMAGE_NAME:$IMAGE_TAG
                '''
               
            }
        }
        stage('Push to ACR') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-creds',
                                                  usernameVariable: 'ACR_USER',
                                                  passwordVariable: 'ACR_PASS')]) {
                    sh '''
                        echo "$ACR_PASS" | docker login $ACR_SERVER -u "$ACR_USER" --password-stdin
                        docker push $ACR_SERVER/$IMAGE_NAME:$IMAGE_TAG
                        docker logout $ACR_SERVER
                    '''
                }
            }
        }
        stage('Create ACR Pull Secret') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr-creds',
                                                  usernameVariable: 'ACR_USER',
                                                  passwordVariable: 'ACR_PASS'),
                                 file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl create secret docker-registry acr-secret \
                            --docker-server=$ACR_SERVER \
                            --docker-username="$ACR_USER" \
                            --docker-password="$ACR_PASS" \
                            -n $K8S_NAMESPACE \
                            --dry-run=client -o yaml | kubectl apply -n $K8S_NAMESPACE -f -
                    '''
                }
            }
        }
        stage('Deploy to AKS') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml
                    '''
                }
            }
        }
    }
         post {
        success {
            script {
                echo "Deployment verified successfully. Sending success email via Brevo API."
                withCredentials([string(credentialsId: 'brevo-api-key', variable: 'BREVO_API_KEY')]) {
                    sh """
                        HTTP_CODE=\$(curl -s -o /tmp/brevo.out -w '%{http_code}' \\
                          -X POST https://api.brevo.com/v3/smtp/email \\
                          -H "api-key: \$BREVO_API_KEY" \\
                          -H "Content-Type: application/json" \\
                          -d '{
                            "sender": {"email": "${EMAIL_FROM}"},
                            "to": [{"email": "${EMAIL_RECIPIENTS}"}],
                            "subject": "SUCCESS: Jenkins Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            "textContent": "Good news!\\n\\nThe pipeline ${env.JOB_NAME} build #${env.BUILD_NUMBER} completed successfully, and the deployment ${DEPLOYMENT_NAME} rolled out successfully to AKS.\\n\\nBuild URL: ${env.BUILD_URL}"
                          }')
                        echo "Brevo responded with HTTP \$HTTP_CODE"
                        cat /tmp/brevo.out || true
                        echo ""
                    """
                }
            }
        }

        failure {
            script {
                echo "Pipeline or deployment verification failed. Sending failure email via Brevo API."
                withCredentials([string(credentialsId: 'brevo-api-key', variable: 'BREVO_API_KEY')]) {
                    sh """
                        HTTP_CODE=\$(curl -s -o /tmp/brevo.out -w '%{http_code}' \\
                          -X POST https://api.brevo.com/v3/smtp/email \\
                          -H "api-key: \$BREVO_API_KEY" \\
                          -H "Content-Type: application/json" \\
                          -d '{
                            "sender": {"email": "${EMAIL_FROM}"},
                            "to": [{"email": "${EMAIL_RECIPIENTS}"}],
                            "subject": "FAILED: Jenkins Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                            "textContent": "The pipeline ${env.JOB_NAME} build #${env.BUILD_NUMBER} FAILED.\\n\\nThis could be due to a build/deploy step failing, or the deployment ${DEPLOYMENT_NAME} failing to roll out successfully in AKS (check the Verify Deployment Rollout stage logs).\\n\\nBuild URL: ${env.BUILD_URL}\\nConsole Log: ${env.BUILD_URL}console"
                          }')
                        echo "Brevo responded with HTTP \$HTTP_CODE"
                        cat /tmp/brevo.out || true
                        echo ""
                    """
                }
            }
        }

        always {
            echo "Build result: ${currentBuild.currentResult}"
        }
    }
}

        
       
    

