pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        ACR_SERVER       = 'democontainerregistry12.azurecr.io'
        IMAGE_NAME       = 'springbootdemo'
        IMAGE_TAG        = 'latest'
       
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
                // archiveArtifacts artifacts: 'trivy-image-report.txt', allowEmptyArchive: true
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
        // stage('Deploy to AKS') {
        //     steps {
        //         withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
        //             sh '''
        //                 kubectl apply -f k8s/deployment.yaml
        //                 kubectl apply -f k8s/service.yaml
        //             '''
        //         }
        //     }
        // }

        }
       
    }

