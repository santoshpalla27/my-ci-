first install the prequties using installation file 

on support server:-

sonarqube go to my account and generate token enter a name and select user token and copy the token make sure token is copied it cant be viewd again and copy the url and token to the pipeline

tomcat main sever should run 8082 so it wont confict with nexus server and should be combined with deploy to war plugin in jenkins copy paste the url of tomcat and tomcat login credentials of the server in the jenkins and use "**.war"" and any name 

nexus server will be running on port 8082 and go to page and create two repos and copy the repo link to the pom.xml and edit maven setting in jenkins server 

jenkins server:- 

install plugins and edit the maven setting file

trivy should be installed on main server and can be run by trivy commands 

docker should be isntalled to build the image and deploy to the tomcat container below 

FROM maven AS buildstage
RUN mkdir /opt/webpage
WORKDIR /opt/webpage
COPY . .
RUN mvn clean install 

FROM tomcat
WORKDIR webapps
COPY --from=buildstage /opt/webpage/target/*.war .
RUN rm -rf ROOT && mv *.war ROOT.war
EXPOSE 8080



Jenkins file 


pipeline {
    agent any

    environment {
        IMAGE_NAME = 'tomcat' // Replace with your Docker image name
        CONTAINER_NAME = 'tomcat-server' // Replace with your container name
    }

    stages {
        stage('Poll SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/santoshpalla27/my-ci-.git'
            }
        }
        stage('sonar analysis') {
            steps {
                sh ''' mvn sonar:sonar \
                      -Dsonar.host.url=http://54.90.107.78:9000/ \
                      -Dsonar.login=squ_cbc28c9f992b252d3ad940fd217c77c3f146f5ec '''
            }
        }
        stage('mvn build') {
            steps {
                sh "mvn clean install "
            }
        }
        stage('Deploy WAR to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://54.90.107.78:8082/')], contextPath: 'webapp', war: '**/*.war'
            }
        }
        stage('mvn deploy to nexus') {
            steps {
                sh "mvn clean deploy "
            }
        }
        stage('Cleanup Old Images') {
            steps {
                sh "docker rmi -f \$(docker images -q --filter 'dangling=true') || true"
                sh "docker rmi -f \$(docker images -q ${IMAGE_NAME}) || true"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME} -f Dockerfile ."
            }
        }
        stage('Scan Docker Image and Repository') {
            steps {
                sh "trivy image ${IMAGE_NAME}"
                sh "trivy fs ."
            }
        }
        stage('Cleanup Old Containers') {
            steps {
                sh "docker rm -f ${CONTAINER_NAME} || true"
            }
        }

        stage('Run New Container') {
            steps {
                sh "docker run -d --name ${CONTAINER_NAME} -p 80:8080 ${IMAGE_NAME}"
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}


