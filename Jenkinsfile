pipeline {

     agent {
                  label ("node1 || node2 ||  node3 || node4 ||  node5 ||  branch ||  main ||  jenkins-node || docker-agent ||  jenkins-docker2 ||  preproduction ||  production")

            }

             environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}
 

     options {
    timeout (time: 1, unit: 'HOURS')
    buildDiscarder(logRotator(numToKeepStr: '2'))
    disableConcurrentBuilds()
    timestamps()
  }       

    stages {

        stage('Setup parameters') {
            steps {
                script {
                    properties([
                        parameters([
                        
                        choice(
                            choices: ['Dev', 'Sanbox', 'Prod'], 
                            name: 'Environment'
                                 
                                ),

                          string(
                            defaultValue: 's4user',
                            name: 'User',
                            description: 'Required to enter your name',
                            trim: true
                            ),

                        string(
                            defaultValue: 's3nivelle',
                            name: 'DB-Tag',
                            description:'Required to enter the image tag',
                            trim: true
                            ),

                        string(
                            defaultValue: 's3nivelle',
                            name: 'UI-Tag',
                            description: 'Required to enter the image tag',
                            trim: true
                            ),
                        string(
                            defaultValue: 's3nivelle',
                            name: 'WEATHER-Tag',
                            description: 'Required to enter the image tag',
                            trim: true
                            ),

                        string(
                            defaultValue: 's3nivelle',
                            name: 'AUTH-Tag',
                            description: 'Required to enter the image tag',
                            trim: true
                            ),

                        ])
                    ])
                }
            }
        }

stage('permission') {
            steps {
                sh '''
cat <<EOF > check.sh
#! /bin/bash 
USER=${USER}
cat permission.txt | grep -i $USER
if 
[[ $? -eq 0 ]]
then 
echo "You have permission to run this job"
else 
echo "You DON'T have permission to run this job"
exit 1
fi 
EOF

 bash check.sh
                '''
            }
        }

        
  
          stage('cleaning') {
            steps {
                echo 'clean the environment'
            }
        }

          stage('SonarQube analysis') {
            agent {
                docker {
                  image 'sonarsource/sonar-scanner-cli:4.7.0'
                }
               }
               environment {
        CI = 'true'
        //  scannerHome = tool 'Sonar'
        scannerHome='/opt/sonar-scanner'
    }
            steps{
                withSonarQubeEnv('Sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('build-dev') {
         when{ 
          expression {
            env.Environment == 'DEV' }
            }
            steps {
                sh '''
cd UI
docker build -t devopseasylearning2021/s4-ui:${BUILD_NUMBER}$UITag .
cd -
cd DB
docker build -t devopseasylearning2021/s4-db:${BUILD_NUMBER}$DBTag .
cd -
cd auth 
docker build -t devopseasylearning2021/s4-auth:${BUILD_NUMBER}$AUTHTag .
cd -
cd weather 
docker build -t devopseasylearning2021/s4-weather:${BUILD_NUMBER}$WEATHERTag .
cd -
                '''
            }
        }

        stage('build-sanbox') {
          when{ 
              expression {
                env.Environment == 'SANBOX' }
                }
            steps {
                sh '''
cd UI
docker build -t devopseasylearning2021/s4-ui:${BUILD_NUMBER}$UITag .
cd -
cd DB
docker build -t devopseasylearning2021/s4-db:${BUILD_NUMBER}$DBTag .
cd -
cd auth 
docker build -t devopseasylearning2021/s4-auth:${BUILD_NUMBER}$AUTHTag .
cd -
cd weather 
docker build -t devopseasylearning2021/s4-weather:${BUILD_NUMBER}$WEATHERTag .
cd -
                '''
            }
        }
        
        stage('build-prod') {
          when{ 
              expression {
                env.Environment == 'PROD' }
                }
            steps {
                sh '''
cd UI
docker build -t devopseasylearning2021/s4-ui:${BUILD_NUMBER}$UITag .
cd -
cd DB
docker build -t devopseasylearning2021/s4-db:${BUILD_NUMBER}$DBTag .
cd -
cd auth 
docker build -t devopseasylearning2021/s4-auth:${BUILD_NUMBER}$AUTHTag .
cd -
cd weather 
docker build -t devopseasylearning2021/s4-weather:${BUILD_NUMBER}$WEATHERTag .
cd -
                '''
            }
        } 

        stage('login') {
            steps {
                sh '''
echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u devopseasylearning2021 --password-stdin
                '''
            }
        }

        stage('push-to-dockerhub-dev') {
          when{ 
              expression {
                env.Environment == 'DEV' }
                }
            steps {
                sh '''
docker push devopseasylearning2021/s4-ui:${BUILD_NUMBER}$UITag 
docker push devopseasylearning2021/s4-db:${BUILD_NUMBER}$DBTag 
docker push devopseasylearning2021/s4-auth:${BUILD_NUMBER}$AUTHTag 
docker push devopseasylearning2021/s4-weather:${BUILD_NUMBER}$WEATHERTag 
                '''
            }
        }

        stage('push-to-dockerhub-sanbox') {
          when{ 
              expression {
                env.Environment == 'SANBOX' }
                }
            steps {
                sh '''
docker push devopseasylearning2021/s4-ui:${BUILD_NUMBER}$UITag 
docker push devopseasylearning2021/s4-db:${BUILD_NUMBER}$DBTag 
docker push devopseasylearning2021/s4-auth:${BUILD_NUMBER}$AUTHTag 
docker push devopseasylearning2021/s4-weather:${BUILD_NUMBER}$WEATHERTag 
                '''
            }
        }

        stage('push-to-dockerhub-prod') {
          when{ 
              expression {
                env.Environment == 'PROD' }
                }
            steps {
                sh '''
docker push devopseasylearning2021/s4-ui:${BUILD_NUMBER}$UITag 
docker push devopseasylearning2021/s4-db:${BUILD_NUMBER}$DBTag 
docker push devopseasylearning2021/s4-auth:${BUILD_NUMBER}$AUTHTag 
docker push devopseasylearning2021/s4-weather:${BUILD_NUMBER}$WEATHERTag 
                '''
            }
        }

        
    stage('update helm charts-dev') {
         when{ 
          expression {
            env.Environment == 'DEV' }
            }
	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 's3nivelle-image', variable: 'IMAGE')
	          ]) {

	            sh '''
                 git config --global user.name "nivellef"
                 git config --global user.email nivellef84@gmail.com
                rm -rf s4-pipeline-practise || true
                git clone  https://nivellef:$IMAGE@github.com/nivellef/s4-pipeline-practise.git
                cd s4-pipeline-practise/CHARTS
cat <<EOF > dev-values.yaml           
        image:
          db:
             repository: devopseasylearning2021/s4-db
             tag: "$DBTag"
          ui:
             repository: devopseasylearning2021/s4-ui
             tag: "$UITag"
          auth:
             repository: devopseasylearning2021/s4-auth
             tag: "$AUTHTag"
          weather:
             repository: devopseasylearning2021/s4-weather
             tag: "$WEATHERTag"
EOF
                git add -A 
                git commit -m "testing jenkins"
                git push https://nivellef:$IMAGE@github.com/nivellef/s4-pipeline-practise.git || true
	            '''
	          }

	        }

	      }

	    }  
           
    stage('update helm charts-sanbox') {
         when{ 
          expression {
            env.Environment == 'SANBOX' }
            }
	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 's3nivelle-image', variable: 'IMAGE')
	          ]) {

	            sh '''
                 git config --global user.name "nivellef"
                 git config --global user.email nivellef84@gmail.com
                rm -rf s4-pipeline-practise || true
                git clone  https://nivellef:$IMAGE@github.com/nivellef/s4-pipeline-practise.git
                cd s4-pipeline-practise/CHARTS
cat <<EOF > sanbox-values.yaml           
        image:
          db:
             repository: devopseasylearning2021/s4-db
             tag: "$DBTag"
          ui:
             repository: devopseasylearning2021/s4-ui
             tag: "$UITag"
          auth:
             repository: devopseasylearning2021/s4-auth
             tag: "$AUTHTag"
          weather:
             repository: devopseasylearning2021/s4-weather
             tag: "$WEATHERTag"
EOF
                git add -A 
                git commit -m "testing jenkins"
                git push https://nivellef:$IMAGE@github.com/nivellef/s4-pipeline-practise.git  || true
	            '''
	          }

	        }

	      }

	    }  

  stage('update helm charts-prod') {
         when{ 
          expression {
            env.Environment == 'PROD' }
            }
	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 's3nivelle-image', variable: 'IMAGE')
	          ]) {

	            sh '''
                 git config --global user.name "nivellef"
                 git config --global user.email nivellef84@gmail.com
                rm -rf s4-pipeline-practise || true
                git clone  https://nivellef:$IMAGE@github.com/nivellef/s4-pipeline-practise.git
                cd s4-pipeline-practise/CHARTS
cat <<EOF > prod-values.yaml           
        image:
          db:
             repository: devopseasylearning2021/s4-db
             tag: "$DBTag"
          ui:
             repository: devopseasylearning2021/s4-ui
             tag: "$UITag"
          auth:
             repository: devopseasylearning2021/s4-auth
             tag: "$AUTHTag"
          weather:
             repository: devopseasylearning2021/s4-weather
             tag: "$WEATHERTag"
EOF
                git add -A 
                git commit -m "testing jenkins"
                git push https://nivellef:$IMAGE@github.com/nivellef/s4-pipeline-practise.git  || true
	            '''
	          }

	        }

	      }

	    }

          stage('wait for argocd') {
            steps {
                echo 'waiting for argocd to detect the change'
            }
        }

    
    }
          post {
   
   success {
      slackSend (channel: '#development-alerts', color: 'good', message: "SUCCESSFUL: Application S4-EKTSS  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

 
    unstable {
      slackSend (channel: '#development-alerts', color: 'warning', message: "UNSTABLE: Application S4-EKTSS  Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }

    failure {
      slackSend (channel: '#development-alerts', color: '#FF0000', message: "FAILURE: Application S4-EKTSS Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    }
   
    cleanup {
      deleteDir()
    }
        }





}
