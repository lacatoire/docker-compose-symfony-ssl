
# ⚡ Symfony Docker-Compose with SSL (MySQL & PostgreSQL) 🐳🔒

This repository demonstrates a project setup using `Docker Compose`, `Symfony`, and secure database interactions with both `MySQL` and `PostgreSQL`, leveraging SSL/TLS connection support.

---

## 🔧 **Prerequisites**

Before running the project, ensure that you have the following installed:
- [🐳 Docker](https://www.docker.com/)
- [📦 Docker Compose](https://docs.docker.com/compose/)

---

## 📂 **Project Structure**

```plaintext
.
├── docker-compose.mysql.yml    # Docker Compose configuration for MySQL
├── docker-compose.postgres.yml # Docker Compose configuration for PostgreSQL
├── app/                        # Symfony project directory
│   ├── .env                    # Symfony environment configuration
│   ├── config/                 # Symfony config files
│   └── ...
├── certs/                      # MySQL SSL certificate directory
│   ├── server-key.pem          # MySQL server private key
│   ├── server-cert.pem         # MySQL server certificate
│   ├── ca-cert.pem             # Certificate Authority (CA) certificate
└── README.md                   # Project documentation
```

---

## ⚙️ **Getting Started**

### 1️⃣ **Clone the repository**

```bash
git clone https://github.com/lacatoire/docker-compose-symfony-ssl
cd docker-compose-symfony-ssl
```  

---

### 2️⃣ **Generate SSL certificates**

#### 🐬 **For MySQL**
Follow these steps to generate self-signed SSL certificates for MySQL:

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

#### 🐘 **For PostgreSQL**
Same process applies, but store the files in the `certs/postgres/` directory.

---

### 3️⃣ **Configure the environment**

Update the `.env` file in the Symfony project:

```dotenv
# For MySQL
DATABASE_URL="mysql://db_user:db_password@database_app:3306/db_name?sslmode=required"

# For PostgreSQL
DATABASE_URL="postgresql://db_user:db_password@database_app:5432/db_name"
```

---

### 4️⃣ **Set up the Docker environment**

Run the following command to build and start the containers:

```bash
docker-compose up --build
```  

This will:
- Start a MySQL/PostgreSQL container with SSL enabled.
- Start a Symfony application container.
### Configure Doctrine

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
For PostGres
```yaml
doctrine:
    dbal:
      driver: 'pdo_pgsql'
      url: '%env(DATABASE_URL)%'
      server_version: '17'
      sslmode: 'verify-ca' # 'verify-full' for production
      sslrootcert: '%env(POSTGRES_SSL_CA)%'
      sslcert: '%env(POSTGRES_SSL_CERT)%'
      sslkey: '%env(POSTGRES_SSL_KEY)%'
```
You will add these variables in your .env
```dotenv
POSTGRES_SSL_KEY=/certs/server-key.pem
POSTGRES_SSL_CERT=/certs/server-cert.pem
POSTGRES_SSL_CA=/certs/ca-cert.pem
```

### 5️⃣ **Test your setup**

#### ✅ **For MySQL**
```bash
docker exec -it database_app mysql -u db_user -p   --ssl-ca=/etc/certs/ca-cert.pem   --ssl-cert=/etc/certs/server-cert.pem   --ssl-key=/etc/certs/server-key.pem db_name
```  

#### ✅ **For PostgreSQL**
```bash
docker exec -it database_app psql -U db_user -d db_name   --set=sslrootcert=/etc/certs/ca-cert.pem   --set=sslcert=/etc/certs/server-cert.pem   --set=sslkey=/etc/certs/server-key.pem
```  
Run the following query to validate SSL:
```sql
SELECT ssl_is_used FROM pg_stat_ssl WHERE pid = pg_backend_pid();
-- Output: 'true' if SSL is enabled
```

#### ✅ **From Symfony**
```bash
docker exec -it php_app php bin/console doctrine:schema:update --dump-sql --complete
```

---

## 🛠 **Troubleshooting**

1. Ensure the SSL certificates are correctly generated and placed in the `certs/` directory.
2. For PostgreSQL, ensure the `sslmode` is set to `verify-ca` (or `verify-full` for production).
3. Check the CN of your server certificate matches `database_app`.

---

## 📜 **License**

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

Interested in training with our team? [Contact us!](https://www.itefficience.com/contact)  

