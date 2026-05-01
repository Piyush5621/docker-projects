
# pull a base image which give all required tols and libraries 
FROM eclipse-temurin:17-jdk-alpine
# create a folder where the app code will be strored
WORKDIR /app

# copy the source code(host main jo rkha huwa ) from host machine to your container
COPY src/Main.java /app/Main.java

# compine the application code
RUN javac Main.java

# run the application 
CMD ["java","Main"]

# ENTRYPOINT [ "executable" ]