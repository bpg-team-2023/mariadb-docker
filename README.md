 `docker-compose.yml` file that sets up a MariaDB 10.3 container with custom networking, a static IP address, and persistent storage for deployment in production.

### `docker-compose.yml`

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.3
    container_name: mariadb-production
    networks:
      mariadb_network:
        ipv4_address: 172.18.0.3
    environment:
      MYSQL_ROOT_PASSWORD: your-root-password
      MYSQL_DATABASE: your-database-name
      MYSQL_USER: your-username
      MYSQL_PASSWORD: your-user-password
    volumes:
      - mariadb_data:/var/lib/mysql
    ports:
      - "3307:3306"  # Mapping internal port 3306 to external port 3307
    restart: always

networks:
  mariadb_network:
    driver: bridge
    ipam:
      driver: default
      config:
        - subnet: 172.18.0.0/16

volumes:
  mariadb_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /opt/mariadb_data
```

### Explanation

1. **Custom Networking**:
   - **Network**: A custom bridge network named `mariadb_network` is created with a specific subnet `172.18.0.0/16`.
   - **Static IP**: The container is assigned a static IP address `172.18.0.3`.

2. **Persistent Storage**:
   - **Volumes**: The data is persisted using a named volume `mariadb_data`. The volume is mapped to the `/var/lib/mysql` directory inside the container, which is where MariaDB stores its data.
   - **Host Directory**: The `device` option in the volume configuration points to a host directory `/opt/mariadb_data`, ensuring that the data persists even if the container is removed or restarted.

3. **Environment Variables**:
   - Configure the MariaDB instance using environment variables such as `MYSQL_ROOT_PASSWORD`, `MYSQL_DATABASE`, `MYSQL_USER`, and `MYSQL_PASSWORD`.

4. **Port Mapping**:
   - In the `ports` section, the line `- "3307:3306"` maps the container's internal port `3306` to the external port `3307`. This means that MariaDB will be accessible on port `3307` of the host machine..

5. **Restart Policy**:
   - The `restart: always` directive ensures that the container automatically restarts in case of failure, making it suitable for production deployment.

6. **Accessing MariaDB from Outside the Host Machine**:
   - **Network Configuration**: Ensure that your server's firewall is configured to allow incoming traffic on port `3307`. Since `firewalld` is not running on your host machine, you can use `iptables` or another firewall tool if necessary.
   - **Public IP Address**: Use the public IP address of your host machine to access the database from another network.
   
   Example command to connect from an external machine:
   ```bash
   mysql -h your-public-ip -P 3307 -u your-username -p

### Steps to Deploy

1. **Save** the `docker-compose.yml` file to a directory on your server.
2. **Create the Persistent Storage Directory**:
   ```bash
   sudo mkdir -p /opt/mariadb_data
   sudo chown -R 1001:1001 /opt/mariadb_data
   ```
   This directory will be used for storing the database files.
3. **Deploy** the MariaDB container:
   ```bash
   docker-compose up -d
   ```
   This command will start the MariaDB container with the specified configuration.

### Accessing MariaDB

- **From the Host Machine**: You can access the MariaDB instance using `localhost` or the server’s IP address:
  ```bash
  mysql -h 127.0.0.1 -P 3306 -u your-username -p
  ```
- **From Another Container**: Use the static IP address `172.18.0.3` to connect from another container on the same network.

4. **Ensure Firewall Rules**: If a firewall is running on your server, add a rule to allow traffic on port `3307`.

### Example Firewall Rule (if needed)

If you need to configure `iptables`, you can add a rule like this:

```bash
sudo iptables -A INPUT -p tcp --dport 3307 -j ACCEPT
```

This command allows incoming TCP connections on port `3307`.

### Summary

- The MariaDB container is now configured to listen on port `3307` externally, while still using port `3306` internally.
- You can access the MariaDB instance from outside the host machine using the host's public IP and port `3307`.
- Make sure your server’s firewall is configured to allow traffic on this port.
- This setup provides a robust and production-ready MariaDB deployment with custom networking, static IP, and persistent storage.