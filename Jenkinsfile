pipeline {
  agent {
         label 'Slave'
    }
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
          maven 'auto_maven'
    }
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
    stages {
        stage('Clean running previous app') {
            steps {
              sh 'docker rm -f pandaapp || true'
            }
        }
        stage('Get Code') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'feature-panda/selenium-grid', credentialsId: 'GitHub', url: 'https://github.com/WitoldSlawko/panda_academy_devops_core_project.git'
            }
        }
        stage('Build and Junit') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Build Docker image') {
            steps {
                sh 'mvn package -Pdocker'
            }
        }
        stage('Run Docker container') {
            steps {
                sh 'docker run -d -p 0.0.0.0:8083:8083 --name pandaapp -t ${IMAGE}:${VERSION}'
            }
        } 
        stage('Test selenium') {
            steps {
                sh 'mvn test -Pselenium'

            }
        }
        stage('Deploy') {
            steps {
               configFileProvider([configFile(fileId: '2a72166b-e9c9-4e14-b87b-2b394ca3fa48', variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                sh 'mvn -gs $MAVEN_GLOBAL_SETTINGS deploy'
              }
            }
        }
        post {
            always {
               sh 'docker stop pandaapp'
               deleteDir()
            }
        }
            // post {
            //     // If Maven was able to run the tests, even if some of the test
            //     // failed, record the test results and archive the jar file.
            //     success {
            //         junit '*/target/surefire-reports/TEST-.xml'
            //         archiveArtifacts 'target/*.jar'
            //     }
            // }
        }
    }
