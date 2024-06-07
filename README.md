## Why we should use docker-compose to build a multi-containers application

Let's consider an *example* of a typical web application that consists of multiple services such as a web server, a database, and a caching service. We would need to manually manage each container separately, ensuring they have the correct configurations and can communicate with each other. This process can be complex and error-prone.

Here's an example of how we *might* write Dockerfiles for a web application, a MySQL database, and a Redis caching service:

1. Web Application Dockerfile (`Dockerfile.web`):
	```dockerfile
	FROM nginx:latest

	WORKDIR /usr/share/nginx/html

	COPY . .

	# Expose port 80 to allow external access
	EXPOSE 80

	# Command to start the nginx server
	CMD ["nginx", "-g", "daemon off;"]
	```
2. MySQL Database Dockerfile (`Dockerfile.db`):
	```dockerfile
	FROM mysql:5.7
	# Environment variables to configure the MySQL database ENV 			  
	MYSQL_ROOT_PASSWORD=password
	# Expose port 3306 to allow external access 
	EXPOSE 3306
	```
3. Redis Caching Service Dockerfile (`Dockerfile.cache`):
	```dockerfile
	# Use an existing Redis image as a base
	FROM redis:alpine

	# Expose port 6379 to allow external access
	EXPOSE 6379
	```

We would need to manually manage networking between containers to ensure they can communicate with each other. This might involve creating custom Docker networks and configuring container links or using IP addresses to establish communication between containers.
```sh
$ docker network create my-network
$ docker run -d --name mysql-db --network my-network -e MYSQL_ROOT_PASSWORD=password mysql:5.7
$ docker run -d --name redis-cache --network my-network redis:alpine
$ docker run -d --name web-app --network my-network -p 8080:80 <our-web-app-image>
```

Now, all three containers are running in the `my-network` Docker network, and they can communicate with each other using their container names as hostnames. For example, the web application container can access the MySQL database using `mysql-db` as the hostname and the Redis caching service using `redis-cache` as the hostname.

## Docker Compose

Without Docker Compose, we would need to manually manage each container separately, ensuring they have the correct configurations and can communicate with each other. This process can be complex and error-prone.

With Docker Compose, we can define the configuration for each service in a single YAML file.

```yaml
# docker-compose.yml
version: '3'

services:
  web:
    build: .
    ports:
      - "8080:80"
    depends_on:
      - db
      - cache
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: example
  cache:
    image: redis:alpine
```

Once we have defined ourr services in the `docker-compose.yml` file, we can easily start our application with a single command:

```sh
$ docker-compose up
```

This command builds the necessary images (if needed), starts the containers, and connects them as defined in the configuration file. And they will be able to communicate with each other within the Docker network created by Docker Compose.

## Why do we should use Docker Compose for multi-containers applications

Docker Compose is used to simplify the process of defining, managing, and running multi-container Docker applications. Here's why it's commonly used:

1. **Orchestration**: Docker Compose allows us to define the configuration for multiple Docker containers within a single YAML file. This makes it easier to manage complex applications composed of multiple interconnected services.

2. **Service Definition**: With Docker Compose, we can specify all the services, networks, and volumes required for our application in one place. This simplifies the deployment process and ensures consistency across different environments.

3. **Easy Deployment**: Docker Compose provides simple commands for building, starting, stopping, and scaling our application. This streamlines the deployment process and reduces the likelihood of errors.

4. **Environment Configuration**: Docker Compose supports environment variables, which allows us to customize the configuration of ourr containers based on the target environment. This makes it easier to deploy our application to different environments such as development, testing, and production.

5. **Isolation**: Each container managed by Docker Compose is isolated from the others, which helps prevent conflicts between different services and dependencies.

6. **Portability**: Docker Compose configurations can be easily shared and reused across different development teams and environments. This promotes collaboration and ensures consistency in deployment practices.

Docker Compose simplifies the management and deployment of multi-container Docker applications, making it a popular choice for developers and DevOps teams.
