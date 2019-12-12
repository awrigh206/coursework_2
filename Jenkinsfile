pipeline {
   agent any

   stages {
      stage('Get files from Github') {
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

        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        /* push the created image onto docker hub so that it can be accessed by other services*/
        docker.withRegistry('https://registry.hub.docker.com/', 'docker-hub-credentials')
		{
            app.push("${env.BUILD_NUMBER}")
            app.push("latest")
        }
    }
	
	withCredentials([sshUserPrivateKey(credentialsId: 'user', keyFileVariable: 'id', passphraseVariable: '', usernameVariable: 'user')]) 
	{
		environment
		{
			def remote = [:]
		  remote.name = 'ansible-node'
		  remote.host = '40.114.47.249'
		  remote.user =$user
		  remote.identity=$id
		  remote.allowAnyHosts = true
		
		
			stage('node SSH') 
			{
				sshCommand remote:remote, command: "kubectl set image deployments/server coursework=awrigh206/coursework:latest"
			}
		}
	}



}
