pipeline {
  agent any

  environment {
    IMAGE = "adyush/python-flask-pipeline"
    IMAGE_TAG = "${BUILD_NUMBER}"
    NAMESPACE = "flask-app"
    K8S_HOST = "k8s-ec2-ssh"
  }

  stages {
    stage("Checkout") {
      steps {
        git branch: "main", url: "https://github.com/adyushomm/python-flask-pipeline.git"
      }
    }

    stage("Build") {
      steps {
        sh "docker build -t ${IMAGE}:${IMAGE_TAG} ."
      }
    }

    stage("Test") {
      steps {
        sh "docker run -d --name flask-test -p 5001:5000 ${IMAGE}:${IMAGE_TAG}"
        sh "sleep 5 && curl --fail http://127.0.0.1:5001/health"
      }
      post {
        always {
          sh "docker rm -f flask-test || true"
        }
      }
    }

    stage("Push") {
      steps {
        withCredentials([usernamePassword(credentialsId: "dockerhub-cred", usernameVariable: "DOCKER_USER", passwordVariable: "DOCKER_TOKEN")]) {
          sh "echo ${DOCKER_TOKEN} | docker login -u ${DOCKER_USER} --password-stdin"
          sh "docker push ${IMAGE}:${IMAGE_TAG}"
        }
      }
    }

    stage("Deploy to Kubernetes") {
      steps {
        sshagent([K8S_HOST]) {
          sh "ssh -o StrictHostKeyChecking=no ubuntu@${K8S_HOST} \"kubectl -n ${NAMESPACE} set image deployment/flask-app flask-app=${IMAGE}:${IMAGE_TAG} && kubectl -n ${NAMESPACE} rollout status deployment/flask-app\""
        }
      }
    }
  }
}
