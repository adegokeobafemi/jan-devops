pipeline {
  agent any
  tools {
  
  maven 'Maven'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'git', url: 'https://github.com/adegokeobafemi/jan-devops.git']]])
        }
      }
	  
	  stage ('Build')  {
	      steps {
            dir('webapp'){
            sh "pwd"
            sh "ls -lah"
            sh "mvn package"
          }
        }
         
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
                
				dir('webapp'){
                 sh 'mvn -U clean install sonar:sonar'
                }
				
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://44.213.18.133:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "jan-devops-libs-release-local",
                    snapshotRepo: "jan-devops-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "jan-devops-libs-release",
                    snapshotRepo: "jan-devops-libs-snapshot"
                )
            }
    }

    stage ('Deploy Artifacts') {
            steps {
                rtMavenRun (
                    tool: "Maven", // Tool name from Jenkins configuration
                    pom: 'webapp/pom.xml',
                    goals: 'clean install',
                    deployerId: "MAVEN_DEPLOYER",
                    resolverId: "MAVEN_RESOLVER"
                )
         }
    }

    stage ('Publish build info') {
            steps {
                rtPublishBuildInfo (
                    serverId: "jfrog"
             )
        }
    }

    stage('Copy Dockerfile & Playbook to Ansible Server') {
            
            steps {
                  ssh_agent(['sshkey']) {
                       
                        sh "scp -o StrictHostKeyChecking=no dockerfile ubuntu@52.202.178.144:/home/ubuntu"
                        sh "scp -o StrictHostKeyChecking=no devops.yaml ubuntu@52.202.178.144:/home/ubuntu"
                    }
                }
            
        } 
    stage('Build Container Image') {
            
            steps {
                 withCredentials([sshUserPrivateKey(credentialsId: "ssh_agent", keyFileVariable: 'keyfile')]){
                       
                        sh "ssh -i ${keyfile} -o StrictHostKeyChecking=no ubuntu@52.202.178.144 -C \"ansible-playbook -i hosts devops.yaml\""
                        
                    }
                }
            
        } 
    stage('Copy Deployent & Service Defination to K8s Master') {
            
            steps {
                  withCredentials([sshUserPrivateKey(credentialsId: "ssh_agent", keyFileVariable: 'keyfile')]){
                       
                        sh "scp -i ${keyfile} -o StrictHostKeyChecking=no deployment.yaml ubuntu@18.206.173.255:/home/ubuntu"
                        sh "scp -i ${keyfile} -o StrictHostKeyChecking=no service.yaml ubuntu@18.206.173.255:/home/ubuntu"
                    }
                }
            
        } 

    stage('Waiting for Approvals') {
            
        steps{

				input('Test Completed ? Please provide  Approvals for Prod Release ?')
			  }
            
    }     
    stage('Copy and Deploy on Production') {
            
            steps {
                  withCredentials([sshUserPrivateKey(credentialsId: "ssh_agent", keyFileVariable: 'keyfile')]){
                        sh"""
                        scp -i ${keyfile} -o StrictHostKeyChecking=no deployment.yaml ubuntu@54.236.58.176:/home/ubuntu
                        scp -i ${keyfile} -o StrictHostKeyChecking=no service.yaml ubuntu@54.236.58.176:/home/ubuntu
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no ubuntu@54.236.58.176 -C \"kubectl apply -f deployment.yaml\"
                        ssh -i ${keyfile} -o StrictHostKeyChecking=no ubuntu@54.236.58.176 -C \"kubectl apply -f service.yaml\"
                        """
                        
                    }
                }
            
        } 
         
   } 
}



