pipeline {
    agent any
    tool{
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
       
    }
}