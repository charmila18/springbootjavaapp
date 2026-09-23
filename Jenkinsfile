pipeline {
    agent any
    tools {
        maven 'maven'
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
                        -Dsonar.organization=charmila18 \
                        -Dsonar.projectKey=springbootapp \
                        -Dsonar.projectName=springbootapp \
                        -Dsonar.java.binaries=target/classes
                        '''
                }
                
            }
        }
       
    }
}