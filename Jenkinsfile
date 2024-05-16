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
    stage('UNIT test & jacoco ') {
          steps {
            sh "mvn test"
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
