
# Symfony Docker-Compose with SSL (MySQL & PostgreSQL)

Docker Compose setup for Symfony with SSL/TLS database connections (MySQL or PostgreSQL).

## Prerequisites

- [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/)

## Quick start

```bash
git clone https://github.com/lacatoire/docker-compose-symfony-ssl
cd docker-compose-symfony-ssl
```

### 1. Generate SSL certificates

Build and run the OpenSSL helper container:

```bash
docker build -t docker-openssl:latest .docker/openssl
docker run -it --rm -v "$PWD/certs:/certs" docker-openssl
```

Inside the container, generate the certificates:

```bash
cd /certs

# Private key
openssl genrsa 2048 > server-key.pem

# CA certificate
openssl req -new -x509 -nodes -days 3650 -key server-key.pem -out ca-cert.pem

# Server certificate
openssl req -new -key server-key.pem -out server-cert.csr
openssl x509 -req -in server-cert.csr -CA ca-cert.pem -CAkey server-key.pem -CAcreateserial -out server-cert.pem -days 3650
```

> For PostgreSQL, store the files in `certs/postgres/`.

### 2. Configure Symfony

Update your `.env`:

```dotenv
# MySQL
DATABASE_URL="mysql://db_user:db_password@database_app:3306/db_name?sslmode=required"
MYSQL_SSL_KEY=/certs/server-key.pem
MYSQL_SSL_CERT=/certs/server-cert.pem
MYSQL_SSL_CA=/certs/ca-cert.pem

# PostgreSQL
DATABASE_URL="postgresql://db_user:db_password@database_app:5432/db_name"
POSTGRES_SSL_KEY=/certs/server-key.pem
POSTGRES_SSL_CERT=/certs/server-cert.pem
POSTGRES_SSL_CA=/certs/ca-cert.pem
```

### 3. Configure Doctrine

**MySQL** — `config/packages/doctrine.yaml`:

```yaml
doctrine:
  dbal:
    driver: 'pdo_mysql'
    url: '%env(resolve:DATABASE_URL)%'
    options:
      !php/const PDO::MYSQL_ATTR_SSL_KEY: '%env(MYSQL_SSL_KEY)%'
      !php/const PDO::MYSQL_ATTR_SSL_CERT: '%env(MYSQL_SSL_CERT)%'
      !php/const PDO::MYSQL_ATTR_SSL_CA: '%env(MYSQL_SSL_CA)%'
      !php/const PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT: false
```

**PostgreSQL** — `config/packages/doctrine.yaml`:

```yaml
doctrine:
  dbal:
    driver: 'pdo_pgsql'
    url: '%env(DATABASE_URL)%'
    server_version: '17'
    sslmode: 'verify-ca'
    sslrootcert: '%env(POSTGRES_SSL_CA)%'
    sslcert: '%env(POSTGRES_SSL_CERT)%'
    sslkey: '%env(POSTGRES_SSL_KEY)%'
```

### 4. Start the containers

```bash
docker-compose up --build
```

### 5. Verify SSL

**MySQL:**
```bash
docker exec -it database_app mysql -u db_user -p \
  --ssl-ca=/etc/certs/ca-cert.pem \
  --ssl-cert=/etc/certs/server-cert.pem \
  --ssl-key=/etc/certs/server-key.pem db_name
```

**PostgreSQL:**
```sql
SELECT ssl_is_used FROM pg_stat_ssl WHERE pid = pg_backend_pid();
```

**Symfony:**
```bash
docker exec -it php_app php bin/console doctrine:schema:update --dump-sql --complete
```

## Troubleshooting

- Verify that SSL certificates are correctly placed in `certs/`.
- For PostgreSQL, use `sslmode: verify-ca` (or `verify-full` in production).
- The CN of your server certificate must match `database_app`.

## License

MIT — See [LICENSE](LICENSE).

Interested in training with our team? [Contact us!](https://www.itefficience.com/contact)

