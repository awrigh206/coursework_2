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
	
	withCredentials([usernamePassword(credentialsId: 'build', passwordVariable: 'password', usernameVariable: 'user')]) 
	{
		environment
		{
			def build = [:]
		  build.name = 'build'
		  build.host = '52.142.24.253'
		  build.user =$password
		  build.password =$user
		  build.allowAnyHosts = true
		  
		  
		  
		  stage('build SSH') 
		  {
			sshCommand remote: build, command: "pwd"

		  }
		  
		}
		
	}
	
	withCredentials([sshUserPrivateKey(credentialsId: 'user', keyFileVariable: 'id', passphraseVariable: '', usernameVariable: '')]) 
	{
		environment
		{
				def node = [:]
			node.name = 'ansible-Node'
			node.host = '40.114.47.249'
			node.user = 'azureuser'
			node.allowAnyHosts = true
			node.identity=$id
		}
		
		stage('node SSH') 
		  {
			sshCommand remote:node, command: "pwd"

		  }
	}



}
