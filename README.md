Ansible Docker Traefik
=========

Ansible role to manage traefik in a docker. 

The main goal is to be able to deploy services with [traefik](https://traefik.io) as a basic reverse proxy that provide tls encryption through [letsencrypt](https://letsencrypt.org/).

This role will generate the docker-compose file, the configurations files and the needed directories in order to deploy Traefik with docker-compose.

Depending on configuration you could either use static definition throught ansible inventories or use dynamic inclusion of docker services through docker labels.

Versions
------------

This version is based on toml files for configuration and work on the 1.7 legacy version of traefik.


Requirements
------------

docker-ce
pip install docker
pip install docker-compose

Role Variables
--------------

View main [role variables](defaults/main.yml).

Define static service configuration with specific variables in your inventory file.

Example:
```
all:
  hosts:
    berliner:
      ansible_host: some-server
      ansible_user: hoauser

  children:
    traefik:
      hosts:
        # localhost:
        berliner:
          ansible_user: hoauser
          traefik_config:
            http_port: "18080"
            logs:
              root: "/tmp/"
            certificates:
              email: "jc+{{ inventory_hostname }}@example.com"
          traefik_services:
            - name: some_docker_service
              backends:
                servers:
                  some_docker_service:
                    url: http://172.18.0.2:8080
              frontends:
                servers:
                  some_docker_service:
                    backend: some_docker_service
                    pass_host_header: yes
                    routes:
                      test0:
                        rule: "Host:some-docker-service.example.com"
```
Configuration is merged with the default [role variables](./defaults/main.yml), see [ansible config](./ansible.cfg) file.

Here we will deploy traefik through docker on **some_server**, there will be one service **some_docker_service**.
Here we have one service called **some_docker_service** running possibly through docker, with the internal IP 172.18.0.2. 
We override some global variables, for example the logs *root* path, so that logs are available in **/tmp/**

Dependencies
------------



Example Playbook
----------------

Configure your inventory to have a list of server under traefik, then run the playbook, see [setup-traefik.yml](./setup-traefik.yml) as an example:

    - hosts: traefik
      name: configure traefik
      roles:
        - ansible-docker-traefik


Development
-----------

### self signed dev certificates

You can work with selfsigned certifcates, you will have to generate them and place them in the root of your playbook.

License
-------

MIT


