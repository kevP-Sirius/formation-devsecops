pipeline {
  agent any

  stages {
      stage('Build Artifact') {
          steps {
            sh 'mvn clean package -DskipTests=true'
            archive 'target/*.jar' //so that they can be downloaded later test aa
          }
      }
       //--------------------------
    //--------------------------
        stage('UNIT test & jacoco ') {
          steps {
            sh "mvn test"
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
            }
          }

        }
    //--------------------------
        stage('Mutation Tests - PIT') {
          steps {
            catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }
          }
            post {
             always {
               pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
             }
           }
        }
    //--------------------------
         stage('Docker Build and Push') {
              steps {
                withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'DOCKER_HUB_PASSWORD')]) {
                  sh 'sudo docker login -u kevinpsirius -p $DOCKER_HUB_PASSWORD'
                  sh 'printenv'
                  sh 'sudo docker build -t kevinpsirius/devops_courses:""$GIT_COMMIT"" .'
                  sh 'sudo docker push kevinpsirius/devops_courses:""$GIT_COMMIT""'
                }
              }
            }
    //--------------------------
     stage('Deployment Kubernetes  ') {
          steps {
            withKubeConfig([credentialsId: 'kube-config']) {
              sh "sed -i 's#replace#kevinpsirius/devops_courses:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
              sh 'kubectl apply -f k8s_deployment_service.yaml'
            }
          }
        }

  }
}
