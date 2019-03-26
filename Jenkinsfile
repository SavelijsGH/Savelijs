def repoName = "spring-petclinic"

pipeline {
    agent { label 'java8' }
    environment{
        // Sonar Project Info - You must replace these vars with the Project Name and Key from the ADOP Portal!
        SONAR_PROJECT_NAME = 'Simple Java project analyzed with the SonarQube Runner'
        SONAR_PROJECT_KEY = 'java-sonar-runner-simple'
        ENVIRONMENT_NAME = 'CI'
        }
    stages{
   stage('Clone Repository') {
   steps {
        // Get some code from a GitHub repository
        git 'https://github.com/denisdbell/spring-petclinic.git'
    
   }}
   
   stage('Run Maven Container') {
       steps {
        //Remove maven-build-container if it exists
        sh " docker rm -f maven-build-container"
        
        //Run maven image
        sh "docker run --rm --name maven-build-container maven-build"
   }}
   
   stage('Deploy Spring Boot Application') {
        steps {
         //Remove maven-build-container if it exists
        sh " docker rm -f java-deploy-container"
       
        sh "docker run --name java-deploy-container --volumes-from maven-build-container -d -p 8080:8080 denisdbell/petclinic-deploy"
   }}
      stage('Clone Repository') {
      steps {
        // Get some code from a GitHub repository
        git 'https://github.com/denisdbell/spring-petclinic.git'
   }}
      stage('Run Maven Container') {
       steps {
        //Remove maven-build-container if it exists
        sh " docker rm -f maven-build-container"
        
        //Run maven image
        sh "docker run --rm --name maven-build-container maven-build"
   }}
   stage('Deploy Spring Boot Application') {
        steps {
         //Remove maven-build-container if it exists
        sh " docker rm -f java-deploy-container"
       
        sh "docker run --name java-deploy-container --volumes-from maven-build-container -d -p 8090:8080 denisdbell/petclinic-deploy"
   }}
}
}