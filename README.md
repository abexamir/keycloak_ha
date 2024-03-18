# Keycloak High Availability Deployment

This project automates the deployment of a highly available Keycloak instance with PostgreSQL as the backend database using Docker containers and Ansible automation.

## Overview

Keycloak is an open-source identity and access management solution that provides single sign-on and identity federation capabilities. In this deployment, Keycloak is configured to run in a highly available setup, ensuring reliability and redundancy.

## Components

- **Keycloak**: The main identity and access management service deployed using Docker containers.
- **PostgreSQL**: The backend database for Keycloak, configured with replication using repmgr for high availability and data redundancy.
- **NGINX**: A reverse proxy in front of Keycloak to handle HTTPS traffic and enforce SSL encryption.
- **Ansible**: Automation tool used to deploy and manage the infrastructure.

## Project Structure

- **docker-compose.j2**: Docker Compose template file for defining the services and their configurations.
- **inventory.yaml**: Ansible inventory file containing host definitions and variables.
- **nginx.conf.j2**: NGINX configuration template for reverse proxying Keycloak.
- **playbook.yml**: Ansible playbook for orchestrating the deployment process.

## Usage

1. Ensure Ansible is installed on your system.
2. Update the `inventory.yaml` file with your host configurations.
3. Customize variables in the `playbook.yml` file according to your environment.
4. Run the Ansible playbook using the command `ansible-playbook playbook.yml`.

## Configuration

- **Custom SSL Certificates**: Enable custom SSL certificates by setting the `use_custom_certs` variable to `"yes"`.
- **Let's Encrypt Integration**: Optionally, configure Let's Encrypt for SSL certificate management by providing an email address in the `letsencrypt_email` variable.
- **Global Domain**: Set the `global_domain` variable to define a global domain name for NGINX configurations if required.

## Integration with Global Load Balancer

To enable users to reach Keycloak using a single point of entry, generate the HAProxy configuration entry for your global load balancer with the following details:

For TCP load balancing mode:
```plaintext
frontend tcp_frontend
    bind *:443
    mode tcp
    option tcplog
    default_backend pg_backend

backend pg_backend
    mode tcp
    balance roundrobin
    server keycloak1 keycloak1.abexamir.me:443 check ssl verify none
    server keycloak2 keycloak2.abexamir.me:443 check ssl verify none
```

For HTTP load balancing mode:
```
frontend http_frontend
    bind *:80
    mode http
    option http-server-close
    redirect scheme https if !{ ssl_fc }

frontend https_frontend
    bind *:443 ssl crt /path/to/your/certificate.pem
    mode http
    option http-server-close
    default_backend pg_backend

backend pg_backend
    mode http
    balance roundrobin
    server keycloak1 keycloak1.abexamir.me:443 check ssl verify none
    server keycloak2 keycloak2.abexamir.me:443 check ssl verify none

```