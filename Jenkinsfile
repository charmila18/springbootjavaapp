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

       
    }
}