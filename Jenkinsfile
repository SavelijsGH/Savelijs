
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
        stage("Reference Application Build"){
            steps{
                echo 'This is a reference Java Application pipeline.'
                deleteDir()
                checkout scmGet("${SCM_URL}", "${SCM_NAMESPACE}", "${repoName}", "${SCM_CREDENTIAL_ID}", 'master')
                sh "./mvnw clean install -DskipTests"
            }


        stage("Reference Application Deploy"){
            steps{
                echo 'This job deploys the java reference application to the CI environment.'
                sh '''
                    set -x
                    export SERVICE_NAME="$(echo ${PROJECT_NAME} | tr \'/\' \'_\')_${ENVIRONMENT_NAME}"
                    docker cp ${WORKSPACE}/target/petclinic.war  ${SERVICE_NAME}:/usr/local/tomcat/webapps/
                    docker restart ${SERVICE_NAME}
                    COUNT=1
                    echo "Count : ${COUNT}"
                    while ! curl -q http://${SERVICE_NAME}:8080/petclinic -o /dev/null
                        do
                            if [ ${COUNT} -gt 10 ]; then
                                echo "Docker build failed even after ${COUNT}. Please investigate."
                                exit 1
                            fi
                            echo "Application is not up yet. Retrying ..Attempt (${COUNT})"
                            sleep 5
                            COUNT=$((COUNT+1))
                        done
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "Environment URL (replace PUBLIC_IP with your public ip address where you access jenkins from) : http://${SERVICE_NAME}.PUBLIC_IP.xip.io/petclinic"
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    set -x
                '''
                stash includes: '**/**', name: 'build-artefacts'
            }
        }

        stage("Reference Application Performance Tests"){
            steps{
                echo 'This job run the Jmeter test for the java reference application.'
                unstash 'build-artefacts'
                sh '''
                    export SERVICE_NAME="$(echo ${PROJECT_NAME} | tr '/' '_')_${ENVIRONMENT_NAME}"
                    if ! grep -e apache-jmeter-2.13.tgz ; then
                        wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-2.13.tgz
                    fi
                    tar -xf apache-jmeter-2.13.tgz
                    echo 'Changing user defined parameters for jmx file'
                    sed -i 's/PETCLINIC_HOST_VALUE/'"${SERVICE_NAME}"'/g' ${WORKSPACE}/src/test/jmeter/petclinic_test_plan.jmx
                    sed -i 's/PETCLINIC_PORT_VALUE/8080/g' ${WORKSPACE}/src/test/jmeter/petclinic_test_plan.jmx
                    sed -i 's/CONTEXT_WEB_VALUE/petclinic/g' ${WORKSPACE}/src/test/jmeter/petclinic_test_plan.jmx
                    sed -i 's/HTTPSampler.path"></HTTPSampler.path">petclinic</g' ${WORKSPACE}/src/test/jmeter/petclinic_test_plan.jmx
                '''
                withAnt(installation: 'ADOP Ant'){
                    sh '''
                        ant -buildfile ${WORKSPACE}/apache-jmeter-2.13/extras/build.xml -Dtestpath=$WORKSPACE/src/test/jmeter -Dtest=petclinic_test_plan
                        export SERVICE_NAME="$(echo ${PROJECT_NAME} | tr '/' '_')_${ENVIRONMENT_NAME}"
                        CONTAINER_IP=$(docker inspect --format '{{ .NetworkSettings.Networks.'"$DOCKER_NETWORK_NAME"'.IPAddress }}' ${SERVICE_NAME})
                        sed -i "s/###TOKEN_VALID_URL###/http:\\/\\/${CONTAINER_IP}:8080/g" ${WORKSPACE}/src/test/gatling/src/test/scala/default/RecordedSimulation.scala
                        sed -i "s/###TOKEN_RESPONSE_TIME###/10000/g" ${WORKSPACE}/src/test/gatling/src/test/scala/default/RecordedSimulation.scala
                        ./mvnw -f ${WORKSPACE}/src/test/gatling/pom.xml gatling:execute
                    '''
                }
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: "${WORKSPACE}/src/test/jmeter/", reportFiles: 'petclinic_test_plan.html', reportName: 'Jmeter Report', reportTitles: ''])
            }
        }
        stage("Reference Application Deploy ProdA"){
            steps{
                echo 'This job deploys the java reference application to the ProdA environment'
                timeout(time:5, unit:'MINUTES') {
                    input('! Deploy to Prod A Environment?')
                }
                sh '''
                    set -x
                    export SERVICE_NAME="$(echo ${PROJECT_NAME} | tr \'/\' \'_\')_PRODA"
                    docker cp ${WORKSPACE}/target/petclinic.war  ${SERVICE_NAME}:/usr/local/tomcat/webapps/
                    docker restart ${SERVICE_NAME}
                    COUNT=1
                    echo "Count : ${COUNT}"
                    while ! curl -q http://${SERVICE_NAME}:8080/petclinic -o /dev/null
                        do
                            if [ ${COUNT} -gt 10 ]; then
                                echo "Docker build failed even after ${COUNT}. Please investigate."
                                exit 1
                            fi
                            echo "Application is not up yet. Retrying ..Attempt (${COUNT})"
                            sleep 5
                            COUNT=$((COUNT+1))
                        done
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "Environment URL (replace PUBLIC_IP with your public ip address where you access jenkins from) : http://${SERVICE_NAME}.PUBLIC_IP.xip.io/petclinic"
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    set -x
                '''
            }
        }
           stage('Run Maven Container') {
       
        //Remove maven-build-container if it exists
        sh " docker rm -f maven-build-container"
        
        //Run maven image
        sh "docker run --rm --name maven-build-container maven-build"
   }
      stage('Deploy Spring Boot Application') {
        
         //Remove maven-build-container if it exists
        sh " docker rm -f java-deploy-container"
       
        sh "docker run --name java-deploy-container --volumes-from maven-build-container -d -p 8090:8080 denisdbell/petclinic-deploy"
   }
        stage("Reference Application Deploy ProdB"){
            steps{
                echo 'This job deploys the java reference application to the ProdB environment'
                timeout(time:5, unit:'MINUTES') {
                    input('! Deploy to Prod B Environment?')
                }
                sh '''
                    set -x
                    export SERVICE_NAME="$(echo ${PROJECT_NAME} | tr \'/\' \'_\')_PRODB"
                    docker cp ${WORKSPACE}/target/petclinic.war  ${SERVICE_NAME}:/usr/local/tomcat/webapps/
                    docker restart ${SERVICE_NAME}
                    COUNT=1
                    echo "Count : ${COUNT}"
                    while ! curl -q http://${SERVICE_NAME}:8080/petclinic -o /dev/null
                        do
                            if [ ${COUNT} -gt 10 ]; then
                                echo "Docker build failed even after ${COUNT}. Please investigate."
                                exit 1
                            fi
                            echo "Application is not up yet. Retrying ..Attempt (${COUNT})"
                            sleep 5
                            COUNT=$((COUNT+1))
                        done
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "Environment URL (replace PUBLIC_IP with your public ip address where you access jenkins from) : http://${SERVICE_NAME}.PUBLIC_IP.xip.io/petclinic"
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    echo "=.=.=.=.=.=.=.=.=.=.=.=."
                    set -x
                '''
            }
        }   
    }
  }
}