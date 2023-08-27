pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK11"
    }

    environment {
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = "your_aws_account.dkr.ecr.us-east-1.amazonaws.com/myprojectimg"
        myprojectRegistry = "https://your_aws_account.dkr.ecr.us-east-1.amazonaws.com"
        cluster = "myprocjectapp-cluster"
        service = "myprojectapp-service"
    }
    stages {
    stage('Fetch code'){
      steps {
        git branch: 'docker', url: 'https://github.com/awsbeginnertutorial/my-project.git'
      }
    }


    stage('Test'){
      steps {
        sh 'mvn test'
      }
    }

    stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('build && SonarQube analysis') {
            environment {
             scannerHome = tool 'sonar4.8'
          }
            steps {
                withSonarQubeEnv('mysonar') {
                 sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=myproject \
                   -Dsonar.projectName=myproject-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

    stage('Build App Image') {
       steps {
       
         script {
                dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
             }

        }
    
    }

    stage('Upload App Image') {
          steps{
            script {
              docker.withRegistry( myprojectRegistry, registryCredential ) {
                dockerImage.push("$BUILD_NUMBER")
                dockerImage.push('latest')
              }
            }
        }
     }

     stage('Deploy to ECS') {
          steps {
            withAWS(credentials: 'awscreds', region: 'us-east-1') {
            sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
        }
      }
     }
  }
}