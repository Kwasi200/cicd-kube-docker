pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "rasbarbies/vproappdock"
        registryCredential = 'docker-hub-credentials-id'
    }

    stages{

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
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

    /*    stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo\
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: True
                }
            }
        }   */


        stage('BUILD APP IMAGE') {
          steps {
            script {
              dockerImage = docker.build registry + ":V$BUILD_NUMBER"
              }
            }

           }

        stage('UPLOAD IMAGE') {
          steps {
            script {
               docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                 dockerImage.push("V$BUILD_NUMBER")
                   dockerImage.push('latest')
               }
            }
          }
         }
        stage('REMOVE UNUSED DOCKER IMAGE') {
          steps{
            sh "docker rmi $registry:V$BUILD_NUMBER"
            }
          }
        stage('CREATE NAMESPACE') {
            steps {
                sh "kubectl create namespace prod || true"
            }
        }
        stage('KUBERNETES DEPLOY') {
          agent {label 'KOPS'}
            steps {
              sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts  --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
              }

        }

    }





}