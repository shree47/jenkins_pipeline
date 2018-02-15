node{


    checkout scm

    def mvnHome


    dir('BuildQuality'){


        stage('Preparation'){


            git 'https://github.com/shree47/simple-spring.git'

            mvnHome = tool 'Maven'


        }

        stage('Build') {

		// Run the maven build
          if (isUnix()) {

                sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"

            } else {

                bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)

            }

        }
		
		 stage('Unit Test Results') {

			try	{
				junit '**/target/surefire-reports/TEST-*.xml'
				//archive 'target/*.jar'
			}
			catch(err)	{
				step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
				throw err
			}
		
		}

        stage('SonarQube Analysis') { 
          // def mvnHome

            //mvnHome = tool 'Maven'

            withSonarQubeEnv('Sonar') { 

                if (isUnix()) {

                    sh "'${mvnHome}/bin/mvn' org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -f pom.xml "+ 

                    " -Dsonar.projectKey=org.sonarqube:java-sonar " +

                    " -Dsonar.projectKey=org.sonarqube:java-sonar " +

                    " -Dsonar.projectName='Java :: Simple Spring Project' " +

                    " -Dsonar.projectVersion=1.0 " +

                    " -Dsonar.language=java " +

                    " -Dsonar.sources=. " +

                    " -Dsonar.tests=. " +

                    " -Dsonar.test.inclusions='**/*Test*/**' " +

					" -Dsonar.exclusions='**/*Test*/**' "


                } else {

                    bat (/"${mvnHome}\bin\mvn" org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -f pom.xml -Dsonar.projectKey=org.sonarqube:java-sonar -Dsonar.projectName="Java :: Simple Spring Project" /)

                }    

            } 


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


stage name:'Deploy to staging', concurrency:1

    node {

                //sh 'sudo docker run -d -p=3000:80 --network=bundlev2_prodnetwork nginx'

        dir('BuildQuality'){

        sh 'sudo docker-compose up -d --build'

		}

	}

node{

   def mvnHome

    dir('FunctionalTests'){

        stage('Get Functional Test Scripts'){                        

		git 'https://github.com/shree47/jenkins-selenium-int-testing.git'

        mvnHome = tool 'Maven'
        
		}

        stage('Run Tests') {

            sh "'${mvnHome}/bin/mvn' -Dgrid.server.url=http://10.0.0.6:4444/wd/hub clean test "

        }

        stage('Functional Test Results') {

            junit '**/target/surefire-reports/TEST-*.xml'
        
		}

    }

}

node ('monitor')	{

	stage('Pull & Run Image to Slave')	{
	
		//sh 'docker login -u="shree47" -p="Jun2015!"' 
		//sh 'docker pull shree47/simple-spring'	
		//sh 'docker run -d --name test shree47/simple-spring'
		
		sh 'docker run -d shree47/simple-spring'
		
		//sh 'docker stop test'
		
		//sh 'docker rm test'
		
		//docker.run("shree47/simple-spring")
	}
}



stage name:'Shutdown staging'

    node {

        dir('BuildQuality'){

        sh 'sudo docker-compose stop'

		}

	}
