# Nextcloud on Raspberry Pi with Docker

This repository provides the steps to install and configure Nextcloud on a Raspberry Pi using Docker and Docker Compose. The setup includes a MariaDB database and an optional reverse proxy for SSL (using Nginx Proxy Manager).

## Prerequisites

- Raspberry Pi (Raspberry Pi 4 recommended)
- Raspberry Pi OS installed
- Docker and Docker Compose installed
- Basic knowledge of terminal commands

## Step 1: Install Docker and Docker Compose

1. Install Docker:

    ```sh
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    ```

2. Install Docker Compose:

    ```sh
    sudo apt install docker-compose -y
    ```

3. Add your user to the Docker group to avoid using `sudo` for Docker commands:

    ```sh
    sudo usermod -aG docker $(whoami)
    ```

4. Log out and log back in, or restart your Raspberry Pi to apply the group changes.

## Step 2: Create a `docker-compose.yml` File

1. Create a directory for Nextcloud:

    ```sh
    mkdir ~/nextcloud-docker
    cd ~/nextcloud-docker
    ```

2. Create a `docker-compose.yml` file with the following content:

    ```yaml
    version: '3'

    services:
      nextcloud:
        image: nextcloud:latest
        container_name: nextcloud
        ports:
          - 8080:80
        volumes:
          - nextcloud_data:/var/www/html
        environment:
          - MYSQL_HOST=db
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud
          - MYSQL_PASSWORD=secret_password
        depends_on:
          - db

      db:
        image: mariadb
        container_name: mariadb
        environment:
          - MYSQL_ROOT_PASSWORD=root_password
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud
          - MYSQL_PASSWORD=secret_password
        volumes:
          - db_data:/var/lib/mysql

    volumes:
      nextcloud_data:
      db_data:
    ```

    - Replace `secret_password` and `root_password` with secure passwords.

## Step 3: Start the Containers
1. Set permissions so everyone can read and write to the mounted partition: 
    ```sh
    sudo chmod -R 777 /mnt/nextcloud
    ```
3. Start the Nextcloud and MariaDB containers using Docker Compose:

    ```sh
    docker-compose up -d
    ```

4. Verify that the containers are running:

    ```sh    
    docker ps
    ```

## Step 4: Configure Nextcloud

1. Open a web browser and navigate to `http://<your-raspberry-pi-ip>:8080`.

2. You will be prompted to create an admin account for Nextcloud. Fill out the necessary details.

3. For the database configuration, use the following details:
    - **Database user**: `nextcloud`
    - **Database password**: the password set in `docker-compose.yml`
    - **Database name**: `nextcloud`
    - **Host**: `db`

4. Complete the setup and start using Nextcloud.

## (Optional) Step 5: Setup SSL with Nginx Proxy Manager

If you want to secure your Nextcloud installation with SSL using Let's Encrypt, you can use Nginx Proxy Manager as a reverse proxy.

1. Add the Nginx Proxy Manager service to your `docker-compose.yml` file:

    ```yaml
    version: '3'

    services:
      nextcloud:
        image: nextcloud:latest
        container_name: nextcloud
        environment:
          - MYSQL_HOST=db
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud
          - MYSQL_PASSWORD=secret_password
        depends_on:
          - db
        volumes:
          - nextcloud_data:/var/www/html
        networks:
          - proxy

      db:
        image: mariadb
        container_name: mariadb
        environment:
          - MYSQL_ROOT_PASSWORD=root_password
          - MYSQL_DATABASE=nextcloud
          - MYSQL_USER=nextcloud
          - MYSQL_PASSWORD=secret_password
        volumes:
          - db_data:/var/lib/mysql
        networks:
          - proxy

      proxy:
        image: jc21/nginx-proxy-manager:latest
        container_name: proxy
        restart: always
        ports:
          - "80:80"
          - "443:443"
          - "81:81"
        volumes:
          - proxy_data:/data
          - proxy_letsencrypt:/etc/letsencrypt

    volumes:
      nextcloud_data:
      db_data:
      proxy_data:
      proxy_letsencrypt:

    networks:
      proxy:
    ```

2. Start all containers:

    ```sh
    docker-compose up -d --build
    ```

3. Access Nginx Proxy Manager at `http://<your-raspberry-pi-ip>:81` and log in with the default credentials.

4. Configure a new proxy host for your domain and enable SSL with Let's Encrypt.

## Access Nextcloud

You can now access your Nextcloud instance through a web browser at:

- `http://<your-raspberry-pi-ip>:8080` (for HTTP)
- `https://<your-domain>` (for HTTPS, if configured with Nginx Proxy Manager)

## License

This project is licensed under the MIT License.

