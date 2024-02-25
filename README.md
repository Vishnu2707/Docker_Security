# Docker_Security
Network Security


To secure a distributed PostgreSQL database system with encryption for data in transit and the potential for encryption for data at rest.

1. Set Up Docker and PostgreSQL Containers**
   - Initialized a Docker environment and set up PostgreSQL containers using `docker-compose`.

2. Configure SSL/TLS for Data in Transit
   - Generate SSL certificates.
   - Configure PostgreSQL to use these certificates for SSL/TLS encrypted connections.

3. Implement Entrypoint Script for PostgreSQL Containers
   - Created and used an entrypoint script to adjust file permissions and ownership within the containers, ensuring PostgreSQL could utilize the provided SSL certificates. (check the main project directory).

5. Tested Database Connectivity
   - Verified the ability to connect to the PostgreSQL database using `psql` with SSL encryption.

-> Commands and Configurations:


1. Docker and PostgreSQL Setup
docker-compose up -d

2. SSL/TLS Configuration
## Generate SSL certificates
sudo openssl genrsa -out server.key 2048
sudo openssl req -new -key server.key -out server.csr -subj "/CN=localhost"
sudo openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
sudo chmod 600 server.key

## Docker Compose volume mounts for SSL certificates (part of docker-compose.yml)
volumes:
  - ./certs/server.crt:/var/lib/postgresql/server.crt
  - ./certs/server.key:/var/lib/postgresql/server.key

## PostgreSQL SSL configuration (part of docker-compose.yml)
command: "-c ssl=on -c ssl_cert_file=/var/lib/postgresql/server.crt -c ssl_key_file=/var/lib/postgresql/server.key"

3. Version Alignment (optional steps based on the chosen method)
## Upgrade psql client to match PostgreSQL server
sudo apt-get update
sudo apt-get install postgresql-client-<version>

4. Entrypoint Script for Container Initialization
## Entrypoint script content (entrypoint.sh)
#!/bin/bash
chown postgres:postgres /var/lib/postgresql/server.key
chmod 600 /var/lib/postgresql/server.key
exec docker-entrypoint.sh postgres

## Make the script executable
chmod +x entrypoint.sh

5. Testing Database Connectivity
psql "sslmode=require host=localhost port=5432 dbname=postgres user=postgres password=mysecretpassword"
```

### To Include: Data at Rest Encryption Using `pgcrypto`

- **Enable `pgcrypto` Extension**
  ```sql
  CREATE EXTENSION IF NOT EXISTS pgcrypto;
  ```

- Inserting and Retrieving Encrypted Data**
  ```sql
  -- Encrypting a value
  INSERT INTO your_table_name (encrypted_column) VALUES (pgp_sym_encrypt('your sensitive data', 'encryption_key_here'));

  -- Decrypting a value
  SELECT pgp_sym_decrypt(encrypted_column, 'encryption_key_here') FROM your_table_name;
  ```
