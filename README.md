# Ansible Project — NGINX Setup for Multiple Environments

This Ansible project installs, configures, and manages NGINX for multiple environments (dev and prod) across different regions (US and WEU).

## Project Structure

```bash
.
├── dev-us-site.yml
├── dev-us-inventory
├── dev-weu-site.yml
├── dev-weu-inventory
├── prod-us-site.yml
├── prod-us-inventory
├── prod-weu-site.yml
├── prod-weu-inventory
├── roles/
│   └── ansible-role-nginx/
│       ├── defaults/
│       │   └── main.yml
│       ├── files/
│       │   └── plqcloudcert.pem
│       ├── handlers/
│       │   └── main.yml
│       ├── tasks/
│       │   └── main.yml
│       ├── templates/
│       │   └── nginx_site.conf.j2
│       ├── vars/
│       │   ├── dev-us.yml
│       │   ├── dev-weu.yml
│       │   ├── prod-us.yml
│       │   ├── prod-weu.yml
├── README.md
```

---

## How It Works

- Each **site file** (e.g., `dev-us-site.yml`) defines the role execution for a specific environment and region.
- Each **inventory file** (e.g., `dev-us-inventory`) defines the list of hosts for that environment.
- Each **vars file** (e.g., `dev-us.yml`) defines environment-specific variables like ports, domains, SSL certificates, and virtual hosts.
- Dynamic vhosts are generated for both HTTP and HTTPS based on the variables inside each environment's vars file.

The site files load the correct vars file depending on which environment/region you are deploying.

---

## Playbook Example

Sample `dev-us-site.yml`:

```yaml
---
- hosts: webservers
  become: yes
  vars_files:
    - roles/ansible-role-nginx/vars/dev-us.yml
  roles:
    - ansible-role-nginx
```

Sample `prod-weu-site.yml`:

```yaml
---
- hosts: webservers
  become: yes
  vars_files:
    - roles/ansible-role-nginx/vars/prod-weu.yml
  roles:
    - ansible-role-nginx
```

---

## Running a Playbook

To deploy NGINX for `dev-us`:

```bash
ansible-playbook -i dev-us-inventory dev-us-site.yml
```

To deploy NGINX for `prod-weu`:

```bash
ansible-playbook -i prod-weu-inventory prod-weu-site.yml
```

---

## Role Details (`ansible-role-nginx`)

### Features

- Installs and configures NGINX
- Enables NGINX to start on boot
- Dynamically configures HTTP and HTTPS vhosts based on environment variables
- Installs SSL certificates for HTTPS
- Validates NGINX configuration (`nginx -t`) **before** applying changes
- Restarts NGINX only if validation passes

### Role Variables

Environment-specific variables include:

- `http_port`: HTTP listening port
- `http_host`: Hostname for HTTP
- `http_post_size`: Max POST body size for HTTP
- `nginx_http_vhost`: List of HTTP virtual hosts with names, addresses, and options

- `https_port`: HTTPS listening port (with `ssl` keyword)
- `https_host`: Hostname for HTTPS
- `ssl_cert`: Path to SSL certificate
- `ssl_key`: Path to SSL private key
- `https_post_size`: Max POST body size for HTTPS
- `nginx_https_vhost`: List of HTTPS virtual hosts with names, addresses, and options

These are defined inside environment-specific vars files like `dev-weu.yml`, `prod-us.yml`, etc.

---

## Handlers

Located in `roles/ansible-role-nginx/handlers/main.yml`:

```yaml
- name: restart nginx
  service:
    name: nginx
    state: restarted
```

Triggered **only when** NGINX configuration or certificates are changed and `nginx -t` passes successfully.

---

## Requirements

- Ansible 2.9 or newer
- Debian or Ubuntu Linux servers
- NGINX official repositories (optional)

---

## Author

Created by **Armughan Bhutta** — [Armughan.Mehboob@PartnerLinQ.com](mailto:Armughan.Mehboob@PartnerLinQ.com)
