def tag = ''
def release = false


pipeline {
  agent {
    kubernetes {
      yamlFile 'agent.yaml'
    }
  }
  environment {
        PROJECT_ID = 'hw-epam-cicd'
        CLUSTER_NAME = 'hw-epam-cicd-gke'
        LOCATION = 'europe-west3-b'
        CREDENTIALS_ID = 'hw-epam-cicd'
  }
  stages {
    stage('Docker Push') {
      steps {
      	container('docker') {
          script {
            if (env.gitlabBranch.contains('refs/tags')) {
              tag = env.gitlabBranch.replace('refs/tags/','')
              release = true
            } else {
              tag = env.BUILD_ID
              release = false
            }
            sh "sed -i 's/__TAG__/${tag}/g' app/templates/index.html"
            docker.withRegistry('https://eu.gcr.io', 'gcr:registry') {
              def image = docker.build("hw-epam-cicd/testapp:${tag}")
              image.push("${tag}")
              if (release) {
                image.push("latest")
              }
            }
          }
	      }
      }
    }
    stage('Deploy') {
      steps{
	      container('kubectl') {
          sh "sed -i 's/__TAG__/${tag}/g' k8s/manifest.yaml"
          step([
            $class: 'KubernetesEngineBuilder',
            projectId: env.PROJECT_ID,
            clusterName: env.CLUSTER_NAME,
            location: env.LOCATION,
            manifestPattern: 'k8s/manifest.yaml',
            namespace: release ? 'test-epam-hw-cicd' : 'dev-epam-hw-cicd',
            credentialsId: env.CREDENTIALS_ID,
            verifyDeployments: true
          ])
	      }
      }
    }
  }
}
