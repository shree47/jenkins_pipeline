node{



    checkout scm

    def mvnHome

    dir('BuildQuality'){

        stage('Preparation'){

                        

            git 'https://github.com/shree47/simple-spring.git'

            mvnHome = tool 'Maven'

        }

	stage('Build Docker image') {


		app=docker.build("shree47/simple-spring")	
	}


	stage('Push to Dockerhub') {

		docker.withRegistry('https://registry.hub.docker.com','shree47'){
		
			app.push("${env.BUILD_NUMBER}")
			app.push("latest")
		}

	}

    
   }
	
}	

stage name:'Shutdown staging'

    node {

                

        dir('BuildQuality'){

        sh 'sudo docker-compose stop'

    }

                

}
