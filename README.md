# docker-wordpress-phpmyadmin
Deploying Wordpress on my local using Docker
Documentation for Docker Compose Configuration
Overview
This docker-compose.yml file defines a multi-container Docker application consisting of three services: a MariaDB database , a phpMyAdmin web interface for managing the database, and a WordPress content management system. The configuration ensures that these services are interconnected and operate seamlessly within a defined network.

Key Components
1. Version
version: "4.3" : Specifies the version of the Docker Compose file format being used. This version supports advanced features such as improved networking and volume management.
2. Services
a) Database Service
image: mariadb:10.6.4-focal : Uses the official MariaDB Docker image version 10.6.4 based on Ubuntu Focal.
mem_limit: 2048m : Limits the memory usage of the container to 2GB.
restart: unless-stopped : Ensures the container restarts automatically unless explicitly stopped.
ports: - 3306:3306 : Maps port 3306 of the host machine to port 3306 of the container, allowing external access to the database.
env_file: .env : Loads environment variables from the .env file.
environment : Defines key environment variables for the database:
MYSQL_ROOT_PASSWORD: Root password for the database (value comes from .env).
MYSQL_DATABASE: Name of the database to create (value comes from .env).
MYSQL_USER: Username for the database (value comes from .env).
MYSQL_PASSWORD: Password for the database user (value comes from .env).
volumes: - db-data:/var/lib/mysql : Persists the database data in a named volume (db-data) to ensure data survival across container restarts.
networks: - wordpress-network : Connects the database service to the wordpress-network.
b) phpMyAdmin Service
depends_on: - database : Ensures that the phpMyAdmin container starts only after the database container is running.
image: phpmyadmin/phpmyadmin : Uses the official phpMyAdmin Docker image.
restart: unless-stopped : Ensures the container restarts automatically unless explicitly stopped.
ports: - 8081:80 : Maps port 8081 of the host machine to port 80 of the container, making phpMyAdmin accessible via http://localhost:8081.
env_file: .env : Loads environment variables from the .env file.
environment : Defines key environment variables for phpMyAdmin:
PMA_HOST: Specifies the hostname of the database server (database in this case).
MYSQL_ROOT_PASSWORD: Root password for the database (value comes from .env).
networks: - wordpress-network : Connects the phpMyAdmin service to the wordpress-network.
c) WordPress Service
depends_on: - database : Ensures that the WordPress container starts only after the database container is running.
image: wordpress:6.2.2-apache : Uses the official WordPress Docker image with Apache version 6.2.2.
restart: unless-stopped : Ensures the container restarts automatically unless explicitly stopped.
ports: - 8080:80 : Maps port 8080 of the host machine to port 80 of the container, making WordPress accessible via http://localhost:8080.
env_file: .env : Loads environment variables from the .env file.
environment : Defines key environment variables for WordPress:
WORDPRESS_DB_HOST: Specifies the hostname and port of the database server (database:3306 in this case).
WORDPRESS_DB_NAME: Name of the database to use (value comes from .env).
WORDPRESS_DB_USER: Username for the database (value comes from .env).
WORDPRESS_DB_PASSWORD: Password for the database user (value comes from .env).
volumes: - ./:/var/www/html : Mounts the current directory (./) on the host machine to the /var/www/html directory inside the container, enabling live updates to WordPress files.
networks: - wordpress-network : Connects the WordPress service to the wordpress-network.
3. Volumes
db-data : A named volume used to persist the database data. This ensures that even if the database container is removed or restarted, the data remains intact.
4. Networks
wordpress-network :
driver: bridge : Creates a custom bridge network named wordpress-network to facilitate communication between the containers (database, phpmyadmin, and wordpress).
How It Works
Database Initialization :
The database service initializes a MariaDB instance using the provided credentials from the .env file.
The database persists its data in the db-data volume.
phpMyAdmin Integration :
The phpMyAdmin service connects to the database service using the hostname database and allows users to manage the database through a web interface accessible at http://localhost:8081.
WordPress Setup :
The wordpress service connects to the database service using the hostname database and the specified database credentials.
WordPress files are mounted from the host machine's current directory, enabling developers to make changes directly in their local environment.
The WordPress site is accessible at http://localhost:8080.
Prerequisites
Docker and Docker Compose : Ensure Docker and Docker Compose are installed on your system.
.env File : Create an .env file in the same directory as the docker-compose.yml file with the following variables:
env
Copy
1
2
3
4
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_DATABASE=your_database_name
MYSQL_USER=your_database_user
MYSQL_PASSWORD=your_database_password
Usage Instructions
Save the docker-compose.yml file and create the .env file with the required variables.
Run the following command to start the services:
bash
Copy
1
docker-compose up -d
Access the services:
WordPress: http://localhost:8080
phpMyAdmin: http://localhost:8081
To stop the services, run:
bash
Copy
1
docker-compose down
Notes
The depends_on directive ensures proper startup order but does not guarantee that the dependent services are fully ready. For critical applications, consider implementing health checks.
The volumes section ensures data persistence for the database and enables live updates for WordPress files.
The networks section facilitates seamless communication between the containers.
