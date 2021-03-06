def imageName = "brunoslalmeida/jenkins-jnlp-slave-dind"

pipeline {
  agent {
    kubernetes {
      label 'jenkins-pod'
    }
  }
  stages {
   
    stage('Build Docker Image') {
      steps {
        script {
          def B_TAG = (env.BRANCH_NAME == "develop" ) ? 'stage' : (env.BRANCH_NAME == "master" ) ? 'prod' : 'latest'
          sh "docker build --network host -t ${imageName}:${B_TAG} ."          
        }
      }
    }

    stage('Creating Release and Tagging') {
      when { 
         not {
          branch 'master';
        }
      }
      steps {
        withCredentials([string(credentialsId: 'petala-gh-token', variable: 'TOKEN')]) {
          sh 'npm install'               
          sh "GH_TOKEN=${TOKEN} node_modules/semantic-release/bin/semantic-release.js"
        }
      }
    }  

    stage('Publish Docker Image') {
      steps {
        script {
          def TAG = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1 | cut -c2-6").trim()
          def TAGA = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1 | cut -c2-4").trim()
          def TAGB = sh(returnStdout: true, script: "git tag --sort version:refname | tail -1 | cut -c2-2").trim()
          
          def B_TAG = (env.BRANCH_NAME == "develop" ) ? 'stage' : (env.BRANCH_NAME == "master" ) ? 'prod' : 'latest'
          
          sh "docker tag ${imageName}:${B_TAG} ${imageName}:${TAG}"
          sh "docker tag ${imageName}:${B_TAG} ${imageName}:${TAGA}"
          sh "docker tag ${imageName}:${B_TAG} ${imageName}:${TAGB}"

          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
            sh "docker login -p ${PASSWORD} -u ${USERNAME} "
          }

          sh "docker push ${imageName}"
        }
      }
    }
  }
}