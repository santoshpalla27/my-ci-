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

go to setting and web hook and add http://<jenkins-domain>/github-webhook/

and content type application Json

and select all events or push by choice and create
