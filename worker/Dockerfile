FROM maven:3.6.1

WORKDIR /app

COPY . .

RUN mvn package -DskipTests &&\
    mv target/worker-jar-with-dependencies.jar /run/worker.jar &&\
    cd /run &&\
    rm -rf /app

CMD java -jar /run/worker.jar
