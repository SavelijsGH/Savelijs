def repoName = "spring-petclinic"

node {

  stages{
   stage('Clone Repository') {
   steps {
        // Get some code from a GitHub repository
        git 'https://github.com/denisdbell/spring-petclinic.git'
    
   }
   }
   
   stage('Run Maven Container') {
       steps {
        //Remove maven-build-container if it exists
        sh " docker rm -f maven-build-container"
        
        //Run maven image
        sh "docker run --rm --name maven-build-container maven-build"
   }
   }
   
   stage('Deploy Spring Boot Application') {
        steps {
         //Remove maven-build-container if it exists
        sh " docker rm -f java-deploy-container"
       
        sh "docker run --name java-deploy-container --volumes-from maven-build-container -d -p 8080:8080 denisdbell/petclinic-deploy"
   }
   }
}
}