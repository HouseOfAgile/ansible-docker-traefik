Ansible Docker Traefik
=========

Ansible role to manage traefik in a docker. 

The main goal is to be able to deploy services with [traefik](https://traefik.io) as a basic reverse proxy that provide tls encryption through [letsencrypt](https://letsencrypt.org/).

This role will generate the docker-compose file, the configurations files and the needed directories in order to deploy Traefik with docker-compose.

Depending on configuration you could either use static definition throught ansible inventories or use dynamic inclusion of docker services through docker labels.

Versions
------------

This playbook is working with latest traefik version 2, and use yaml file for configuration (see [traefik migration v1 to v2](https://docs.traefik.io/migration/v1-to-v2/)).

If you want to use an old traefik version you can check the [traefik-v1.7](https://github.com/HouseOfAgile/ansible-docker-traefik/tree/traefik-v1.7) branch version

Principles
------------

In order to have an efficient reverse proxy with ssl encryptions, traefik is a great solution, this playbook ease the maintenance of your services and traefik definitions. 

This traefik installation is by default supposed to work with docker (and you can disbale it), so if you set your labels properly on your docker containers, they should be taken into account by traefik. 
You can also work with static definition, as long as you are within the same network, you should have your services properly mapped with ssl encryption.

By default, there is a global middleware to reirect traefik from http to https, this part from traefik is a bit hacky and should evolve in the future, as many discussions are ongoing on how to do it properly.

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
    some_server:
      ansible_host: some-server
      ansible_user: hoauser

  children:
    traefik:
      hosts:
        # localhost:
        some_server:
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
                        rule: "Host(`some-docker-service.example.com`)"
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

You can work with self signed certifcates, you will have to generate them and place them in the root of your playbook.

License
-------

MIT


