# Deploy SearXNG Using Ansible

This guide outlines the necessary steps to deploy **SearXNG**, a privacy-respecting metasearch engine, using **Ansible** on a server.

## Prerequisites

- A server running **Ubuntu** or a compatible Debian-based system.
- SSH access to the server.

## Installation Steps

1. **Update and upgrade the system packages then install Ansible**:

``` bash
sudo apt update && sudo apt upgrade -y && sudo apt install ansible -y

```
   
2. **Create the Ansible Playbook Files**:

• Create a file named deploy_searxng.yml and populate it with the following content:
   

``` yaml
---
- name: Deploy SearXNG on Docker
  hosts: target_server  # Replace with your inventory or group name
  become: true

  vars:
    searxng_port: "8888"
    base_url: "http://localhost:8888"
    docker_network_name: "searxng-net"

  tasks:
    - name: Install Docker
      apt:
        name: docker.io
        state: present
        update_cache: true

    - name: Ensure Docker is started
      service:
        name: docker
        state: started
        enabled: true

    - name: Pull the SearXNG Docker image
      docker_image:
        name: searxng/searxng
        source: pull

    - name: Create Docker network (optional)
      docker_network:
        name: "{{ docker_network_name }}"
        state: present

    - name: Run the SearXNG container
      docker_container:
        name: searxng
        image: searxng/searxng
        state: started
        ports:
          - "{{ searxng_port }}:8080"
        env:
          BASE_URL: "{{ base_url }}"
        networks:
          - name: "{{ docker_network_name }}"
        restart_policy: always

```

• Create an inventory file named inventory.ini and add the following content:

``` ini
[target_server]
localhost ansible_connection=local
# Replace 'localhost' with the target server's IP or hostname for remote deployment.
```


## **Running the Ansible Playbook**


1. **Navigate to the directory containing the Ansible files**:

``` bash
cd /path/to/your/ansible/directory
```


2. **Run the playbook**:

``` bash
ansible-playbook -i inventory.ini deploy_searxng.yml
```


**Customization**

• Modify the vars section in deploy_searxng.yml to change the searxng_port, base_url, or docker_network_name as needed.

• To run the playbook with different variable values, use the -e flag:

``` bash
ansible-playbook -i inventory.ini deploy_searxng.yml -e "searxng_port=9999 base_url=http://example.com"
```


**Conclusion**

Following these steps, you should have **SearXNG** deployed and running on your server using Ansible. Enjoy a privacy-respecting metasearch experience!

