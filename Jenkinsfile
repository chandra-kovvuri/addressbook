pipeline {
    agent none


    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven"
    }
    environment{
        BUILD_SERVER='ec2-user@172.31.92.116'
        IMAGE_NAME='vikranth2009/devops-learning'
        DEPLOY_SERVER='ec2-user@172.31.39.67'
    }
    stages {
        stage('compile') {
            agent {label "Jenkins_Node1"}
            //agent any
            steps {
                script{

                    // Run Maven on a Unix agent.
                    git 'https://github.com/chandra-kovvuri/addressbook.git'
                    echo "Compiling the code in this stage"
                    sh "mvn compile"
                }               
            }
        }

        stage('test') {  
            agent any
            steps {
                script{
        
                    echo "Testing the code in this stage."
                    sh "mvn test"
                }            
            }
            post{ 
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Containerise-Build-Docker-Image') {
            agent any
            steps {
                script{
                    sshagent(['Jenkins_Slave2_SSh_Key']) {
                        withCredentials([usernamePassword(credentialsId: 'docker-hub-Jenkins-Credentials', passwordVariable: 'docker-hub-password1', usernameVariable: 'docker-hub-username1')]) {   
                        sh "scp -o StrictHostKeyChecking=no containerise-docker-build.sh ${BUILD_SERVER}:/home/ec2-user"
                        sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/containerise-docker-build.sh ${IMAGE_NAME} ${BUILD_NUMBER}'"   
                        sh "ssh ${BUILD_SERVER} sudo docker login -u ${docker-hub-username} -p ${docker-hub-password}"
                        sh "ssh ${BUILD_SERVER} sudo docker push ${IMAGE_NAME}:${BUILD_NUMBER}"
                              
                    }
                    }   

                }
            }
        }
        stage ('Deploy the docker container'){
            agent any
            steps{
                script{
                    sshagent(['Jenkins_Slave2_SSh_Key']) {

                        withCredentials([usernamePassword(credentialsId: 'docker-hub-Jenkins-Credentials', passwordVariable: 'docker-hub-password1', usernameVariable: 'docker-hub-username1')]) {    
                        sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} sudo yum install docker -y"
                        sh "ssh ${BUILD_SERVER} sudo systemctl start docker"
                        sh "ssh ${BUILD_SERVER} sudo docker login -u '${docker-hub-username}' -p '${docker-hub-password}'"
                        sh "ssh ${BUILD_SERVER} sudo docker run -itd -P ${IMAGE_NAME} ${BUILD_NUMBER}"

                       }   
                    }    
                                 

                }
                
            }

        }
    }
}