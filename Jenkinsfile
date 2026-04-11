pipeline {
  agent any

  parameters {
    string(name: "App_Version", description: "Provide application version")
  }

  environment {
    DOCKERHUB_CREDENTIALS = credentials("dockerhub")
  }

  stages {

    stage("Checkout") {
      steps {
        git branch: 'main', url: 'https://github.com/amit6264/DataStore-project.git'
      }
    }

    stage("Check Workspace") {
      steps {
        sh '''
          echo "----- WORKSPACE FILES -----"
          pwd
          ls -R
        '''
      }
    }

    stage("Maven Build") {
      steps {
        dir('DataStore-project') {
          sh '''
            echo "-------- Building Application --------"
            mvn clean package
            echo "------- Application Built Successfully --------"
          '''
        }
      }
    }

    stage("Maven Test") {
      steps {
        dir('DataStore-project') {
          sh '''
            echo "-------- Executing Testcases --------"
            mvn test
            echo "-------- Testcases Execution Complete --------"
          '''
        }
      }
    }

    stage("Artifact Store") {
      steps {
        dir('DataStore-project') {
          sh '''
            echo "-------- Uploading Artifact To S3 --------"
            aws s3 cp ./target/*.jar s3://datastore-artefact-store-jenkins-apps-amit/
            echo "-------- Artifact Upload Completed --------"
          '''
        }
      }
    }

    stage("Docker Image Build") {
      steps {
        dir('DataStore-project') {
          sh '''
            echo "-------- Building Docker Image --------"
            docker build -t datastore:"${App_Version}" .
            echo "-------- Docker Image Built Successfully --------"
          '''
        }
      }
    }

    stage("Docker Image Scan") {
      steps {
        dir('DataStore-project') {
          sh '''
            echo "-------- Scanning Docker Image --------"
            trivy image datastore:"${App_Version}"
            echo "-------- Image Scan Completed --------"
          '''
        }
      }
    }

    stage("Docker Image Tag") {
      steps {
        dir('DataStore-project') {
          sh '''
            echo "-------- Tagging Docker Image --------"
            docker tag datastore:"${App_Version}" amitpatidar/datastore-repo:"${App_Version}"
            echo "-------- Tagging Completed --------"
          '''
        }
      }
    }

    stage("Docker Login & Push") {
      steps {
        dir('DataStore-project') {
          sh '''
            echo "-------- Logging Into DockerHub --------"
            docker login -u $DOCKERHUB_CREDENTIALS_USR --password $DOCKERHUB_CREDENTIALS_PSW
            echo "-------- Login Successful --------"

            echo "-------- Pushing Docker Image --------"
            docker push amitpatidar/datastore-repo:"${App_Version}"
            echo "-------- Docker Image Pushed Successfully --------"
          '''
        }
      }
    }

    stage("Cleanup") {
      steps {
        sh '''
          echo "-------- Cleaning Up Jenkins Machine --------"
          docker image prune -a -f
          echo "-------- Cleanup Completed --------"
        '''
      }
    }

    stage("Deployment Acceptance") {
      steps {
        input 'Trigger Downstream Job?'
      }
    }
  }
}
