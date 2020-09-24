def awsCredentials = [[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-personal']]


pipeline {
  agent {
         label 'Slave'
    }
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
          maven 'auto_maven'
          terraform 'Terraform'
    }
    environment {
        IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
        ANSIBLE = tool name: 'Ansible', type: 'com.cloudbees.jenkins.plugins.customtools.CustomTool'
    }
    options {
        withCredentials(awsCredentials)
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
                // git branch: 'feature-panda/selenium-grid', credentialsId: 'GitHub', url: 'https://github.com/WitoldSlawko/panda_academy_devops_core_project.git'
                checkout scm
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
        stage('Run terraform') {
            steps {
                dir('infrastructure/terraform') {                
                    sh 'terraform init && terraform apply -var-file ./varsenv.tfvars -auto-approve '
                } 
            }
        }
        stage('Copy Ansible role') {
               steps {
                   sh 'cp -r infrastructure/ansible/panda/ /etc/ansible/roles/'
                }
        }
        stage('Run Ansible') {
               steps {
                dir('infrastructure/ansible') {                
                    sh 'chmod 400 ../panda-new-keys.pem'
                    sh 'ansible-playbook -i ./inventory playbook.yml'
                } 
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
        post {
               always {
                 sh 'docker stop pandaapp'
                 deleteDir()
              }
           }

    }
