pipeline {
  agent any
  triggers {
    pollSCM '* * * * *'
  }
  stages {
    stage('SonarQube Analysis') {
      steps {
        sh '''
	 whoami
	 echo $PATH
         echo Restore started on `date`.
         dotnet restore panz.csproj
         dotnet build panz.csproj -c Release
        
        '''
      }
    }
    stage('Dotnet Publish') {
      steps {
        sh 'dotnet publish panz.csproj -c Release'
      }   
    }
   stage('Docker build and push') {
      steps {
        sh '''
         whoami
	 echo $access_key
	 echo $secret_key
         aws configure set aws_access_key_id $access_key
         aws configure set aws_secret_access_key $secret_key
         aws configure set default.region ap-south-1
         DOCKER_LOGIN_PASSWORD=$(aws ecr get-login-password  --region ap-south-1)
         docker login -u AWS -p $DOCKER_LOGIN_PASSWORD https://522726045829.dkr.ecr.ap-south-1.amazonaws.com
	 docker build -t 522726045829.dkr.ecr.ap-south-1.amazonaws.com/demo-${BUILD_NUMBER} .
	 docker push 522726045829.dkr.ecr.ap-south-1.amazonaws.com/demo-PROJECT-${BUILD_NUMBER}
          
	  '''
     }   
   }
    stage('ecs deploy') {
      steps {
        sh '''
          chmod +x changebuildnumber.sh
          ./changebuildnumber.sh $BUILD_NUMBER
	  sh -x ecs-auto.sh
          '''
     }    
    }
}
post {
    failure {
        mail to: 'unsolveddevops@gmail.com',
             subject: "Failed Pipeline: ${BUILD_NUMBER}",
             body: "Something is wrong with ${env.BUILD_URL}"
    }
     success {
        mail to: 'unsolveddevops@gmail.com',
             subject: "successful Pipeline:  ${env.BUILD_NUMBER}",
             body: "Your pipeline is success ${env.BUILD_URL}"
    }
}

}
