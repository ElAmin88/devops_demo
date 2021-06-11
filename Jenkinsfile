

pipeline {
  agent any
  tools {
  maven 'maven'
  }

    stages {

      stage ('Checkout SCM'){

        steps {

          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/ElAmin88/devops_demo.git']]])

        }

      }

      stage ('Build')  {

          steps {

            sh "cd  java-source"

            sh "mvn package"

          }

      }

     stage ('SonarQube Analysis') {

        steps {

              withSonarQubeEnv('sonarqube') {

                sh "cd  java-source"

        sh 'mvn -U clean install sonar:sonar'

              }

            }

      }

    stage ('Artifactory configuration') {

            steps {

                rtServer (
                    id: "jfrog",
                    url: "http://10.0.2.20:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (

                    id: "MAVEN_DEPLOYER",
                    serverId: "artifactory",
                    releaseRepo: "devops-libs-release-local",
                    snapshotRepo: "libs-snapshot"

                )

                rtMavenResolver (

                    id: "MAVEN_RESOLVER",

                    serverId: "artifactory",

                    releaseRepo: "devops-libs-release-local",

                    snapshotRepo: "libs-snapshot"

                )

            }

    }

    stage ('Deploy Artifacts') {

            steps {

                rtMavenRun (

                    tool: "maven", // Tool name from Jenkins configuration

                    pom: 'java-source/pom.xml',

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


    stage('Build Container Image') {

            steps {

                  sshagent(['sshkey']) {

                        sh "ssh -o StrictHostKeyChecking=no admin@10.0.2.30 -C \"sudo ansible-playbook create-container-image.yml\""

                    }
                }
        } 



    stage('Waiting for Approvals') {

        steps{
                input('Test Completed ? Please provide  Approvals for Prod Release ?')
              }
    }     

    stage('Deploy Artifacts to Production') {
            steps {
                  sshagent(['sshkey']) {

                        sh "ssh -o StrictHostKeyChecking=no admin@10.0.2.20 -C \"sudo kubectl apply -f tomecat_deployment.yaml.yaml\""

                        sh "ssh -o StrictHostKeyChecking=no admin@10.0.2.20 -C \"sudo kubectl apply -f nodeport.yaml\""

                    }
                }
        } 
   } 
}

