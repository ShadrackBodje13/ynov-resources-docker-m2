# SUBMISSION.md

## Group members :

    - Shadrack BODJE
    - Boutaina BELGHITI
    - Ibrahim CHOUAY

## Etapes :

### Ceate a work directory in your vritual machine

### Fork link Github : 

#### Git clone into your machine (https://github.com/ShadrackBodje13/ynov-resources-docker-m2/tree/main/2023/m2/dataeng/humans-best-friend)

### Build images with docker-compose.build.yml : 

$vi docker-compose.build.yml

```
version: '3'
services:
  worker:
    build:
      context: ./worker
    depends_on:
      redis:
        condition: service_healthy
      db:
        condition: service_healthy
    networks:
      - humansbestfriend-network

  vote:
    build:
      context: ./vote
    volumes:
      - ./vote:/usr/local/app
    ports:
      - "5002:80"
    networks:
      - humansbestfriend-network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 15s
      timeout: 5s
      retries: 3
      start_period: 10s

  seed-data:
    build:
      context: ./seed-data
    depends_on:
      vote:
        condition: service_healthy
    restart: "no"
    networks:
      - humansbestfriend-network

  result:
    build:
      context: ./result
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./result:/usr/local/app
    ports:
      - "5001:80"
      - "127.0.0.1:9229:9229"
    networks:
      - humansbestfriend-network

  db:
    image: postgres:15-alpine
    volumes:
      - "db-data:/var/lib/postgresql/data"
      - "./healthchecks:/healthchecks"
    healthcheck:
      test: /healthchecks/postgres.sh
      interval: "5s"
    networks:
      - humansbestfriend-network

  redis:
    image: redis
    networks:
      - humansbestfriend-network
    volumes:
      - "./healthchecks:/healthchecks"
    healthcheck:
      test: /healthchecks/redis.sh
      interval: "5s"

networks:
  humansbestfriend-network:
    external: true

volumes:
  db-data:
    external: false

```

#### Execute file build : $docker compose -f docker-compose.build.yml build

### Create registry :
$docker run -d -p 5000:5000 --restart always --name registry registry:2

#### Tag images created with docker-compose.build.yml : 

$docker tag worker:latest localhost:5000/worker:latest
$docker tag vote:latest localhost:5000/vote:latest
$docker tag seed-data:latest localhost:5000/seed-data:latest
$docker tag result:latest localhost:5000/result:latest

#### Push these images : 

$docker push localhost:5000/worker:latest
$docker push localhost:5000/vote:latest
$docker push localhost:5000/seed-data:latest
$docker push localhost:5000/result:latest

#### Verify that all images are added to the registry : 

$curl localhost:5000/v2/_catalog ( you can replace `localhost` with your virtual machine`s address IP) 

Check if you can access the link `localhost:5000/v2/_catalog` on your navigator 

#### Create compose.yml : we used the images in the registry to fill up compose.yml
```
version: '3'
services:
  vote:
    image: localhost:5000/vote
    ports:
      - "5001:80"
  result:
    image: localhost:5000/result
    ports:
      - "5002:80"
  seed-data:
    image: localhost:5000/seed-data
  worker:
    image: localhost:5000/worker

```

### Execute docker compose with : `$docker compose up -d`

## Open the links : 

### For interface Vote : `localhost:5001`
### For interface Result : `localhost:5002`



