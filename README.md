# ⚡ Symfony MySQL Docker-Compose with SSL 🐳🔒

This repository demonstrates a project setup using `Docker Compose`, `Symfony`, and `MySQL` with SSL/TLS connection support for secure database interactions.
## Prerequisites

Before running the project, ensure that you have the following installed:
- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)

## Project Structure

```directory
.
├── docker-compose.yml      # Docker Compose configuration
├── app/                    # Symfony project directory
│   ├── .env                # Symfony environment configuration
│   ├── config/             # Symfony config files
│   └── ...
├── certs/                  # MySQL SSL certificate directory
│   ├── server-key.pem      # MySQL server private key
│   ├── server-cert.pem     # MySQL server certificate
│   ├── ca-cert.pem         # Certificate Authority (CA) certificate
└── README.md               # Project documentation
```

## ⚙️ Getting Started
### 1. Clone the repository

```bash
  git clone https://github.com/your_username/symfony-mysql-ssl-docker.git
  cd symfony-mysql-ssl-docker
```

### 2. Generate SSL certificates for MySQL

If you don't already have SSL certificates, you can generate self-signed certificates using OpenSSL. Run the following commands to generate your certificates:

```bash
# Build Docker image
docker build -t docker-openssl:latest .docker/openssl

# Run Docker container in interactive mode
# Make sure you replace `<your_path>` with your target folder, this is where files will be created.

# For Docker Desktop (Windows Pro)
docker run -it --rm -v "C:\your_path\certs:/certs" docker-openssl

# For Docker Toolbox (Windows Home/linux/mac)
docker run -it --rm -v "$YOUR_PATH/certs:/certs" docker-openssl
```
And then once connected
```bash
mkdir -p certs
cd certs
    
# Generate private key
openssl genrsa 2048 > server-key.pem
    
# Generate Certificate Authority (CA) certificate
openssl req -new -x509 -nodes -days 3650 -key server-key.pem -out ca-cert.pem
    
# Generate server certificate
openssl req -new -key server-key.pem -out server-cert.csr
openssl x509 -req -in server-cert.csr -CA ca-cert.pem -CAkey server-key.pem -CAcreateserial -out server-cert.pem -days 3650
```

### 3. Configure the environment

Edit the .env file in the Symfony project directory to set the database connection details `SSL Mode`

```bash
DATABASE_URL="mysql://db_user:db_password@database_app:3306/db_name?sslmode=required"
```
### 4. Set up the Docker environment

Run the following command to build and start the containers:

```bash
docker-compose up --build
```
This will:

    Start a MySQL container with SSL enabled.
    Start a Symfony application container.

### 5. Configure Doctrine

In the config/packages/doctrine.yaml file, add the SSL options for MySQL:

```yaml

doctrine:
  dbal:
    driver: 'pdo_mysql'
    url: '%env(resolve:DATABASE_URL)%'
    options:
      1007: '%env(MYSQL_SSL_KEY)%' 
      ## or !php/const PDO::MYSQL_ATTR_SSL_KEY: '%env(MYSQL_SSL_KEY)%'
      
      1008: '%env(MYSQL_SSL_CERT)%'
      ## or !php/const PDO::MYSQL_ATTR_SSL_CERT: '%env(MYSQL_SSL_CERT)%'
            
      1009: '%env(MYSQL_SSL_CA)%'
      ## or !php/const PDO::MYSQL_ATTR_SSL_CA: '%env(MYSQL_SSL_CA)%'
      
      1014: false
      ## or !php/const PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT: false
```
You will add these variables in your .env
```dotenv
MYSQL_SSL_KEY=/certs/server-key.pem
MYSQL_SSL_CERT=/certs/server-cert.pem
MYSQL_SSL_CA=/certs/ca-cert.pem
```


### 6. Access the Symfony application

After the containers have been started, tests your ssl connection on mysql with

```bash
docker exec -it database_app mysql -u db_user -p --ssl-ca=/etc/certs/ca-cert.pem --ssl-cert=/etc/certs/server-cert.pem --ssl-key=/etc/certs/server-key.pem db_name
```

And then test from your symfony app:
```bash
docker exec -it php_app php bin/console doctrine:schema:update --dump-sql --complete
```

### Troubleshooting
If you encounter connection issues with MySQL SSL, ensure the following:

    The SSL certificates are correctly generated and placed in the certs directory.
    The CN of the MySQL server certificate matches the hostname database_app as expected.
    Ensure Docker Compose is using the correct volumes and environment settings.

### License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

Interested in training with our team? [Contact us](https://www.itefficience.com/contact)!
