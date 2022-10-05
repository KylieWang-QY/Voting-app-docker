# Voting App
This app is based on [Docker Samples/example-voting-app](https://github.com/dockersamples/example-voting-app), a simple distributed application runing across multiple Docker containers.
## Architecture
![Figure: Architecture](/architecture.png)
- A front-end web app in Python which lets you vote between two options
- A redis queue which collects new votes
- A .NET Core processor which consumes votes and stores them in ...
- A Postgres database backed by a Docker volume
- A Node.js web app which shows the results of the voting in real time
- The app will be runing at [http://localhost:5000](http://localhost:5000), and the results will be at [http://localhost:5001](http://localhost:5001)
## Getting started
Download [Docker Desktop](https://www.docker.com/products/docker-desktop/) for Mac. 
## Deploying application - Docker build
1. Build image in the vote file: voting app
```
docker build . -t voting-app
```
2. Run container: voting-app. Check the [http://localhost:5000](http://localhost:5000), the voting interface will be displayed  
```
docker run -p 5000:80 voting-app
```
3. Run container: redis and link to voting app. Restart voting-app container and check the [http://localhost:5000](http://localhost:5000) again, the voting results will be displayed on the terminal.
```
docker run -d --name=redis redis
docker run -p 5000:80 --link redis:redis voting-app
```
4. Run container: db (Postgres:9.4)*
```
docker run -d --name=db -e POSTGRES_HOST_AUTH_METHOD=trust postgres:9.4
```
5. Build image in the worker file: worker-app
```
docker build . -t worker-app
```
6. Run container: worker-app, link to db and redis
```
docker run --link redis:redis --link db:db worker-app
```
7. Build image in the result file: result-app*
```
docker build . -t result-app
```
8. Run container: result-app, link to db. Make sure all containers restarted, and check [http://localhost:5000](http://localhost:5000) and [http://localhost:5001](http://localhost:5001), the voting application will run as expected. 
```
docker run -p 5001:80 --link db:db result-app
```
## Notes
- For step 4, should add `-e POSTGRES_HOST_AUTH_METHOD=trust` to allow all connections without a password. Although, this is not recommended. Another method to add `POSTGRES_USER` and `POSTGRES_PASSWORD` to the environment variable, I use this method in the docker-compose. Please check [ref](https://stackoverflow.com/questions/63262613/docker-postgres-database-is-uninitialized-and-superuser-password-is-not-specif) for more information.
- For step 7, should add `--platform=linux/amd64` to the FROM line of the docker file to avoid error: "qemu-x86_64: Could not open '/lib64/ld-linux-x86-64.so.2': No such file or directory". Please check [ref](https://stackoverflow.com/questions/71040681/qemu-x86-64-could-not-open-lib64-ld-linux-x86-64-so-2-no-such-file-or-direc) for more information.

## Deploying application - Docker-compose
1. Check docker-compose version
```
docker-compose --version
```
2. Stop all the running containers
3. Use docker-compose.yaml to deploy the application, and check [http://localhost:5000](http://localhost:5000) and [http://localhost:5001](http://localhost:5001), the voting application will run as expected. 
```
docker-compose up
```