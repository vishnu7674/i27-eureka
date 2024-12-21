# official openjdk 
FROM openjdk:18

# create the directory /opt/i27 , inwhich we store our application
RUN mkdir -p /opt/i27/

# setting the working directory to /opt/i27
WORKDIR /opt/i27/

#Define a build type variable called jar_source to specify the location of jar file
ARG JAR_SOURCE

#copy the JAR file
COPY ["${JAR_SOURCE}", "/opt/i27/i27cart-eureka.jar"]

#Grant RWX permissons on this directory
RUN chmod 777 /opt/i27/

#expose port 8761 to allow external access to application
EXPOSE 8761

# Specify the command to run the spring based applications using the JAVA JAR file

CMD ["java", "-jar", "/opt/i27/i27cart-eureka.jar"]

