pipeline {
    agent { label 'docker-maven-slave' } 
    triggers {
          pollSCM('4 4 4 * *')
    }
    environment {
          VER_NUM = "1.0.${BUILD_NUMBER}";
          REL_NUM = "1.0.${BUILD_NUMBER}.RELEASE";
          mavenHome =  tool name: "maven", type: "maven"
     }
    tools{
          maven 'maven'
     }
    stages {
           stage ('Git Checkout') {
                 steps {
                     echo "Heloo!!!!";
                     git credentialsId: 'github-credentials' , url: 'https://github.com/bathuru/simpleapp.git',  branch: 'master'   
                }
           }

         stage ('Multiple Builds') {
              parallel {
                  stage ("Maven Build") {
                        steps {
                             echo "Hello 222";
                            //sh "${mavenHome}/bin/mvn clean versions:set -Dver=${VER_NUM} package "
                            sh "${mavenHome}/bin/mvn clean package "
                       }
                  }
                  stage ("Gradel Build") {
                        steps {
                            echo "Gradel Build !!!!!!!"
                       }
                  }
              }
          }
    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog_server",
                    url: "https://bathuru.jfrog.io/artifactory",
                    credentialsId: "jfrog_cred"
                )
                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog_server",
                    releaseRepo: "simpleapp-local",
                    snapshotRepo: "simpleapp-local"
                )
                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog_server",
                    releaseRepo: "default-maven-virtual",
                    snapshotRepo: "default-maven-virtual"
                )
                rtMavenRun (
                    tool: "maven", // Tool name from Jenkins configuration
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
                rtPublishBuildInfo (
                    serverId: "jfrog_server"
             )
         }
            }

     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar_server') {
                   sh '${mavenHome}/bin/mvn verify org.sonarsource.scanner.maven:sonar-maven-plugin:sonar -Dsonar.projectKey=simpleapp'
                 //sh "${mavenHome}/bin/mvn sonar:sonar"
              }
            }
      }    
 
          stage('Docker Build & Push') {    
                  steps {
                          script{        // To add Scripted Pipeline sentences into a Declarative
                                    try{
                                            sh "pwd"
                                             //sh "docker rm -f simpleapp || true"
                                             //sh "docker rmi bathurudocker/simpleapp || true"       //sh 'docker rmi $(docker images bathurudocker/simpleapp)''
                                          }catch(error){
                                          //  do nothing if there is an exception
                                          }
                            }
                          //sh "docker build -t bathurudocker/devops-simpleapp:${VER_NUM} ."
                          sh "docker build -t bathurudocker/devops-simpleapp:latest ."
                          //sh "docker image tag bathurudocker/devops-simpleapp:${VER_NUM}  bathurudocker/devops-simpleapp:latest"
                          withCredentials([string(credentialsId: 'dockerHubPwd', variable: 'dockerpwd')]) {
                                 sh "docker login -u bathurudocker -p ${dockerpwd}"
                         }
                          //sh "docker push bathurudocker/devops-simpleapp:${VER_NUM}" 
                          sh "docker push bathurudocker/devops-simpleapp:latest" 
                          sh "docker rmi bathurudocker/devops-simpleapp" 
                 } 
          }

     stage('Deploy Into DEV') {
       steps {   
           sh "pwd"
           sshagent(['aws-ap-south-pem']) {
               sh "ssh -o StrictHostKeyChecking=no ec2-user@docker.bathur.xyz  sudo docker rm -f bathurudocker/devops-simpleapp || true"
               //sh "ssh -o StrictHostKeyChecking=no ec2-user@docker.bathur.xyz  sudo docker run  -d -p 80:8080 --name simpleapp bathurudocker/devops-simpleapp:${VER_NUM}"
              sh "ssh -o StrictHostKeyChecking=no ec2-user@docker.bathur.xyz  sudo docker run  -d -p 80:8080 --name devops-simpleapp bathurudocker/devops-simpleapp:latest"
          }
       }
     }     

         stage('Deploy Into PROD') {
             steps {  
           sh "pwd"
           sshagent(['aws-ap-south-pem']) {
               sh "scp -o StrictHostKeyChecking=no simpleapp-deploy.yaml  simpleapp-ingress-rules.yaml  simpleapp-playbook-k8s.yml ec2-user@ansible.bathur.xyz:/home/ec2-user/"
               sh "ssh -o StrictHostKeyChecking=no ec2-user@ansible.bathur.xyz   ansible-playbook  -i /etc/ansible/hosts /home/ec2-user/simpleapp-playbook-k8s.yml"
          }
             }
     }    
         stage('Build Helm Charts') {
            steps {
              dir('charts') {
             sh "/usr/local/bin/helm package iwayq-web-app"
					   sh "sudo /usr/local/bin/helm push-artifactory --username prreddy1986@gmail.com --password mko09ijN iwayq-web-app-0.0.1.tgz https://iwayqweb.jfrog.io/artifactory/iwayq"
					  }
          }
        } 
        
    }
    post {
           success {
                echo 'Pipeline Sucessfully Finished'
 /*               mail bcc: '', 
                body: """ Hi Team, 
                Your project Build and Deployed successfully.

Please find the details as below,
	   Job Name: ${env.JOB_NAME}
	   Job URL : ${env.JOB_URL}
      Build Number: ${env.BUILD_NUMBER} 
      Build URL: ${env.BUILD_URL}

Thanks
DevOps Team""", 
                          cc: '', 
                          from: '', 
                          replyTo: '', 
                          subject: "${env.JOB_NAME} - Build # ${env.BUILD_NUMBER} - Sucess !!!", 
                          to: 'srinivas.bathuru@gmail.com'  */
           }
           failure {
                echo 'Pipeline Failure'
           }

/*
           always {
emailext attachLog: true, 
 body: '''Hi Team, 
	   
Your project Build and Deployed successfully.

Please find the details as below,
	   Job Name: $PROJECT_NAME 
	   Job URL : $BUILD_URL
       Build Number : $BUILD_NUMBER
       Build Status : $BUILD_STATUS
	   
Thanks
DevOps Team 2''', 
     subject: '$BUILD_STATUS - $PROJECT_NAME - Build # $BUILD_NUMBER ', 
             to: 'srinivas.bathuru@gmail.com'
           }  */
    }
}
