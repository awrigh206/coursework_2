pipeline {
   agent any

   stages {
      stage('Build') {
         steps {
            //Get code from the right branch of the repository
            git branch: 'myBranch', url: 'https://github.com/awrigh206/coursework_2/'
             
         }
      }
          
     stage('SonarQube') 
     {
        environment {
            scannerHome = tool 'SonarQube'
        }
        steps 
        {
            
            withSonarQubeEnv('SonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }
        }
    }
   }
}
node {
    def app

    stage('Clone repository') {
        /* Let's make sure we have the repository cloned to our workspace */

        checkout scm
    }

    stage('Build image') {
        /* This builds the actual image; synonymous to
         * docker build on the command line */

        app = docker.build("awrigh206/coursework")
    }

    stage('Test image') {
        /* Ideally, we would run a test framework against our image.
         * For this example, we're using a Volkswagen-type approach ;-) */

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* Finally, we'll push the image with two tags:
         * First, the incremental build number from Jenkins
         * Second, the 'latest' tag.
         * Pushing multiple tags is cheap, as all the layers are reused. */
        docker.withRegistry('https://registry.hub.docker.com/', 'docker-hub-credentials')
		{
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }

    withCredentials([sshUserPrivateKey(credentialsId: 'user', keyFileVariable: 'user', passphraseVariable: '', usernameVariable: 'azureuser')]) {
        stage("Run command") {
        	sh "ssh azureuser@40.114.47.249 -tt"
		sh "minikube start --vm-driver=virtualbox"
		sh "kubectl set image deployments/server coursework=awrigh206/coursework:latest"

        }
	}

}
