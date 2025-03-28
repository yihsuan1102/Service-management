## Overview

This guide outlines a comprehensive hands-on project designed to simulate real-world scenarios in deploying and managing containerized services across multiple Linux virtual machines. The project provides step-by-step instructions over several days, guiding participants through the setup, configuration, and orchestration of services such as GitLab, Mattermost, Nginx, and Portainer using Docker and Docker Swarm.

Participants will gain practical experience in:

- Installing and configuring Linux environments
- Deploying services with Docker and Docker Compose
- Managing containers with Portainer
- Setting up reverse proxies and HTTPS using Nginx and Certbot
- Automating deployments with Ansible
- Implementing distributed storage using NFS
- Scaling services and handling node failures in a Docker Swarm cluster
- Using mDNS for hostname-based communication in local networks
- Automating OS installation with Ubuntu’s autoinstall

  
## Security Configuration
Disable Linux security settings and firewalls, including `ufw`, `firewalld`, `selinux`, and `apparmor`.

- ufw: `sudo systemctl disable --now ufw`
- firewalld: `sudo systemctl disable --now firewalld`
- selinux: [Disable SELinux guide](https://docs.oracle.com/en/storage/storage-software/storagetek-tape-analytics/2.4/stais/disable-selinux.html)
- apparmor: `sudo systemctl disable apparmor`

Windows firewalls may also block incoming connections. Add the virtualization software to the firewall exceptions list if necessary.

---

## Day 1 – Environment Preparation

- Install a Linux virtual machine and set up Git and Docker.
- Ensure the `git` command works for cloning a repository.
- Verify Docker works by running `docker run hello-world`.

### GitLab Container
- Run GitLab container using [GitLab's official Docker instructions](https://docs.gitlab.com/ee/install/docker.html) with image `gitlab/gitlab-ce:17.0.3-ce.0`.
- Set `external_url` to `http://<your_ip>:8000` and map port `8000:8000`.
- Initialize GitLab, set root password to `___`, and create a repo named `test` using the root account.
- Confirm access to GitLab's web interface from an external network.

---

## Day 2 – Mattermost and Nginx

### Mattermost Container
- Deploy Mattermost without the included NGINX, following [Mattermost's Docker guide](https://docs.mattermost.com/install/install-docker.html).
- Set `.env` variables:
  - `MM_SERVICESETTINGS_SITEURL=http://<your_ip>/`
  - `DOMAIN=<your_ip>`
- Set admin password to `___`, use port `8065`, and confirm external web access.

### Certbot
- Get a domain name and issue an SSL certificate with Certbot container ([guide](https://github.com/mattermost/docker/blob/main/docs/issuing-letsencrypt-certificate.md)).
- Set up port forwarding for port 80 if required.
- Delete the Certbot container after obtaining the certificate.

### Nginx Container
- Run Nginx container with volumes for SSL certificates:
  - -v "${PWD}/certs/etc/letsencrypt:/etc/letsencrypt" -v "${PWD}/certs/lib/letsencrypt:/var/lib/letsencrypt"
- Use the following configuration:
```nginx
server {
    listen 443 ssl;
    server_name <your_domain_name>;

    ssl_certificate /etc/letsencrypt/live/<your_domain_name>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<your_domain_name>/privkey.pem;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```
- Use port mapping 443:443 and confirm HTTPS access to Nginx welcome page.

## Day 3 – Docker Compose and Portainer

### Docker Compose

- Stop GitLab, Mattermost, and Nginx containers.
- Re-deploy GitLab and Nginx using Docker Compose, ensuring all volumes are preserved.

### Upgrade Containers

- Upgrade GitLab to `gitlab/gitlab-ce:17.1.1-ce.0`.
- Upgrade Mattermost to `mattermost/mattermost-enterprise-edition:8.1.13`.
- Verify that all features function correctly after updates.

### Portainer

- Deploy Portainer using Docker Compose ([official guide](https://docs.portainer.io/start/install-ce/server/docker/linux)), setting admin password to `DbseSummer2024`.
- Use Portainer to start, stop, and inspect all Docker containers, view logs and statistics.
- Run the `hello-world` container and verify its logs contain "Hello from Docker".

### Additional Virtual Machine

- Set up a second Linux VM with Git and Docker installed for use in Day 4.

---

## Day 4 – Container Migration

### Method 1: Backup and Restore with Docker Compose

- Backup container volumes from the first VM and shut it down.
- Transfer backups to the second VM and re-deploy GitLab, Mattermost, and Nginx using Docker Compose.
- Verify that all containers start correctly and data is intact.

### Method 2: NFS Shared Folder

- Set up an NFS server on the first VM and export the directories containing container data.
- On the second VM, mount the NFS exports and use them as volumes for Docker containers.
- Re-deploy GitLab, Mattermost, and Nginx on the second VM.
- Confirm service functionality and data persistence.

---

## Day 5 – Automated Installation and Ansible

### OS Auto-installation

- Use the [Ubuntu auto-installation guide](https://canonical-subiquity.readthedocs-hosted.com/en/latest/intro-to-autoinstall.html) to deploy a third VM with the same environment.
- Burn the modified ISO to a USB using Rufus, edit `grub.cfg`, and add the `autoinstall.yaml` configuration file.
- Use default credentials: username `___`, password`___` .

### Ansible Automation

- With VM 1 and VM 3 running, use Ansible to:
  - Mount the NFS share on VM 3.
  - Adjust Docker Compose configurations if needed.
  - Start GitLab, Mattermost, and Nginx containers using the NFS-mounted volumes.
- Verify that all services work correctly and delete VM 3 if successful.

---

## Day 6 – mDNS and Swarm

### mDNS

- Install a fourth VM with hostname `dbsecloud`, and install the `avahi-daemon` package.

### Ansible with mDNS

- Update the Ansible inventory to use `dbsecloud.local` instead of a static IP address.
- Run the same playbook used for VM 3 to mount NFS and start containers via Docker Compose.

### Docker Swarm Setup

- Review the [Docker Swarm overview](https://docs.docker.com/engine/swarm/) and [stack deployment guide](https://docs.docker.com/engine/swarm/stack-deploy/).
- Initialize a Docker Swarm cluster using VM 1, VM 2, and VM 4.
- Deploy GitLab, Mattermost, and Nginx as Docker Swarm services.
- Install Portainer in Swarm mode ([Swarm guide](https://docs.portainer.io/start/install-ce/server/swarm/linux)) and complete initial setup.

---

## Days 7–9 – Final Exercise

### Objective

Act as the Lab's system administrator and complete the following multi-node Docker infrastructure setup:

1. Install three fresh Linux VMs named `dbse1`, `dbse2`, and `dbse3`.
2. Form a Docker Swarm cluster using the three VMs.
3. Deploy GitLab, Mattermost, Nginx, and Portainer. Ensure all services are publicly accessible, with Nginx served via HTTPS.
4. Simulate hardware failure by shutting down `dbse3`, and confirm all services remain functional.
5. Simulate a traffic spike by launching 10 additional Nginx containers. Confirm all services stay responsive.
6. Restart `dbse3` and confirm that it re-joins the Swarm and all services remain operational.
7. Launch an additional Mattermost container for heavy API usage, reusing the existing PostgreSQL service.
8. Add a fourth VM `dbse4` to the Swarm, remove `dbse2`, shut it down, and confirm no service disruption.

---

## Reflection Questions

1. What are the pros and cons of using Ubuntu's autoinstall method versus post-install configuration using Ansible?
2. How does shutting down a Swarm node differ from properly removing it from the Swarm first?
3. What happens if the NFS server storing container volumes fails? How does it impact service availability?
4. How should `apt upgrade` and other system updates be managed, considering they may require a reboot?
5. If a GitLab container is compromised and root access is gained, can it access data from other containers? What about Portainer?
