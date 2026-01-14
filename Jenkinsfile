
pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: kaniko-build
spec:
  serviceAccountName: jenkins
  containers:
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command: ['cat']
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ['cat']
    tty: true
  volumes:
  - name: docker-config
    projected:
      sources:
      - secret:
          name: dockerhub-creds
          items:
          - key: .dockerconfigjson
            path: config.json
"""
    }
  }

  environment {
    IMAGE = "docker.io/sunilmeena26/hello-app"
    TAG = "v${BUILD_NUMBER}"
    DEPLOYMENT = "hello-app"
    NAMESPACE = "default"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/sunilmeena26/Docker-Practice2.git'
      }
    }

    stage('Build & Push Docker Image') {
      steps {
        container('kaniko') {
          sh """
            /kaniko/executor \
              --context="${WORKSPACE}" \
              --dockerfile="${WORKSPACE}/Dockerfile" \
              --destination="${IMAGE}:${TAG}" \
              --skip-tls-verify
          """
        }
      }
    }

    stage('Deploy to GKE') {
      steps {
        container('kubectl') {
          sh """
            kubectl -n ${NAMESPACE} set image deployment/${DEPLOYMENT} ${DEPLOYMENT}=${IMAGE}:${TAG} --record
            kubectl -n ${NAMESPACE} rollout status deployment/${DEPLOYMENT}
          """
        }
      }
    }
  }

  post {
    success {
      echo "New version deployed: ${IMAGE}:${TAG}"
    }
  }
}
