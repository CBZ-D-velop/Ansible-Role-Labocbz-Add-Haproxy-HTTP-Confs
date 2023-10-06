# Ansible role: labocbz.add_haproxy_http_confs

![Licence Status](https://img.shields.io/badge/licence-MIT-brightgreen)
![CI Status](https://img.shields.io/badge/CI-success-brightgreen)
![Testing Method](https://img.shields.io/badge/Testing%20Method-Ansible%20Molecule-blueviolet)
![Testing Driver](https://img.shields.io/badge/Testing%20Driver-docker-blueviolet)
![Language Status](https://img.shields.io/badge/language-Ansible-red)
![Compagny](https://img.shields.io/badge/Compagny-Labo--CBZ-blue)
![Author](https://img.shields.io/badge/Author-Lord%20Robin%20Cbz-blue)

## Description

![Tag: Ansible](https://img.shields.io/badge/Tech-Ansible-orange)
![Tag: Debian](https://img.shields.io/badge/Tech-Debian-orange)
![Tag: HAProxy](https://img.shields.io/badge/Tech-HAProxy-orange)
![Tag: HTTP](https://img.shields.io/badge/Tech-HTTP-orange)
![Tag: Load balancer](https://img.shields.io/badge/Tech-Load%20balancer-orange)

An Ansible role create and add HAProxy HTTP confs to your server.

The role is designed to create HAProxy configurations tailored for HTTP/HTTPS services. It operates through a set of YAML-based configuration objects, providing extensive flexibility in defining various aspects of the HAProxy setup.

The administrator can easily specify the frontend and backend configurations for different websites using the role's parameters. Each website can be assigned a unique name, and its corresponding frontend can be configured with essential details like descriptions, bind addresses, ports, and whether HTTPS is enabled.

For the backend, the role offers the ability to choose the load balancing mode (e.g., round-robin, least connections), set various options (e.g., httpclose, forwardfor, httpchk), define the forwarded port (commonly 443 for HTTPS), and set the desired HTTP check status.

Additionally, administrators can specify multiple servers within a backend, each representing a separate backend server that HAProxy will forward requests to. For each server, the role allows the specification of names, addresses, ports, and whether HTTPS is enabled. Moreover, a backup mode can be enabled for certain servers to ensure fault tolerance.

In summary, the role complements an existing HAProxy installation by enabling the creation of customized configurations for HTTP/HTTPS services. With its flexible YAML-based approach, administrators can easily define various parameters, including ports, SSL settings, load balancing options, and backend server configurations. The role provides a powerful solution for tailoring HAProxy configurations to meet the specific requirements of different websites and services.

## Folder structure

By default Ansible will look in each directory within a role for a main.yml file for relevant content (also man.yml and main):

```PYTHON
.
├── README.md  # Contains an overview of the role and its purpose.
├── defaults
│   ├── main.yml  # Contains default variables for the role that can be overridden by users.
│   └── README.md  # Contains documentation for the default variables.
├── files
│   └── README.md  # Contains documentation for the files in the directory.
├── handlers
│   ├── main.yml  # Contains handlers that can be called by tasks within the role.
│   └── README.md  # Contains documentation for the handlers.
├── meta
│   ├── main.yml  # Contains metadata about the role, including dependencies and supported platforms.
│   └── README.md  # Contains documentation for the role metadata.
├── tasks
│   ├── main.yml  # Contains tasks to be executed by the role on the managed nodes.
│   └── README.md  # Contains documentation for the tasks.
├── templates
│   └── README.md  # Contains documentation for the templates.
└── vars
    ├── main.yml  # Contains variables that are specific to the role and are not meant to be overridden.
    └── README.md  # Contains documentation for the role variables.
```

## Tests and simulations

### Basics

You have to run multiples tests. *tests with an # are mandatory*

```MARKDOWN
# lint
# syntax
# converge
# idempotence
# verify
side_effect
```

Executing theses test in this order is called a "scenario" and Molecule can handle them.

Molecule use Ansible and pre configured playbook to create containers, prepare them, converge (run the role) and verify its execution.
You can manage multiples scenario with multiples tests in order to get a 100% code coverage.

This role contains a ./tests folder. In this folder you can use the inventory or the tower folder to create a simualtion of a real inventory and a real AWX / Tower job execution.

### Command reminder

```SHELL
# Check your YAML syntax
yamllint -c ./.yamllint .

# Check your Ansible syntax and code security
ansible-lint --config=./.ansible-lint .

# Execute and test your role
molecule lint
molecule create
molecule list
molecule converge
molecule verify
molecule destroy

# Execute all previous task in one single command
molecule test
```

## Installation

To install this role, just copy/import this role or raw file into your fresh playbook repository or call it with the "include_role/import_role" module.

## Usage

### Vars

Some vars a required to run this role:

```YAML
---
haproxy_confs_path: "/etc/haproxy/conf.d"
haproxy_ssl_path: "/etc/haproxy/ssl"

haproxy_configurations:
  - name: "my.website.domain.tld"
    frontend:
      description: "My first website  with HTTP frontend address"
      bind: "*"
      port: 10030
      https: false
      mode: "http"
    backend:
      balance: roundrobin
      options:
        - httpclose
        - forwardfor
        - httpchk GET /
      forwardedPort: 443
      httpCheckStatus: 301
      servers:
        - name: "backend-server-1"
          addresse: "backend1.internal.domain.tld"
          port: "80"
          https: false

  - name: "my.https.website.domain.tld"
    frontend:
      description: "My website but with HTTPS frontend access"
      bind: "*"
      port: 10031
      https: true
      mode: "http"
    backend:
      balance: leastconn
      options:
        - httpclose
        - forwardfor
        - httpchk GET /
      forwardedPort: 443
      httpCheckStatus: 200
      servers:
        - name: "backend-server-1-as-BACKUP"
          addresse: "backend1.internal.domain.tld"
          port: 443
          https: true
          backup: true
        - name: "backend-server-2"
          addresse: "backend2.internal.domain.tld"
          port: 443
          https: true

```

The best way is to modify these vars by copy the ./default/main.yml file into the ./vars and edit with your personnals requirements.

You can set vars in the template model in Ansible AWX / Tower or just surchage them during the playbook call.

In order to surchage vars, you have multiples possibilities but for mains cases you have to put vars in your inventory and/or on your AWX / Tower interface.

```YAML
# From inventory
---
inv_haproxy_confs_path: "/etc/haproxy/conf.d"
inv_haproxy_ssl_path: "/etc/haproxy/ssl"

inv_haproxy_configurations:
  - name: "my.website.domain.tld"
    frontend:
      description: "My first website  with HTTP frontend address"
      bind: "*"
      port: 10030
      https: false
      mode: "http"
    backend:
      balance: roundrobin
      options:
        - httpclose
        - forwardfor
        - httpchk GET /
      forwardedPort: 443
      httpCheckStatus: 503
      servers:
        - name: "backend-server-1"
          addresse: "127.0.0.1"
          port: "8181"
          https: true

  - name: "my.https.website.domain.tld"
    frontend:
      description: "My HTTPS website but with 1 server as backup"
      bind: "*"
      port: 10031
      https: true
      mode: "http"
    backend:
      balance: leastconn
      options:
        - httpclose
        - forwardfor
        - httpchk GET /
      forwardedPort: 443
      httpCheckStatus: 503
      servers:
        - name: "backend-server-1-as-BACKUP"
          addresse: "127.0.0.1"
          port: 8181
          https: true
          backup: true
        - name: "backend-server-2"
          addresse: "127.0.0.1"
          port: 8181
          https: true

```

```YAML
# From AWX / Tower
---

```

### Run

To run this role, you can copy the molecule/default/converge.yml playbook and add it into your playbook:

```YAML
- name: "Include labocbz.add_haproxy_http_confs"
  tags:
    - "labocbz.add_haproxy_http_confs"
  vars:
    haproxy_confs_path: "{{ inv_haproxy_confs_path }}"
    haproxy_configurations: "{{ inv_haproxy_configurations }}"
    haproxy_ssl_path: "{{ inv_haproxy_ssl_path }}"
  ansible.builtin.include_role:
    name: "labocbz.add_haproxy_http_confs"
```

## Architectural Decisions Records

Here you can put your change to keep a trace of your work and decisions.

### 2023-05-03: First Init

* First init of this role with the bootstrap_role playbook by Lord Robin Crombez

### 2023-05-27: Bind / SSL

* Added a bind address
* Added SSL/TLS handling

### 2023-05-30: Cryptographic update

* SSL/TLS Materials are not handled by the role
* Certs/CA have to be installed previously/after this role use

### 2023-10-06: New CICD, new Images

* New CI/CD scenario name
* Molecule now use remote Docker image by Lord Robin Crombez
* Molecule now use custom Docker image in CI/CD by env vars
* New CICD with needs and optimization

## Authors

* Lord Robin Crombez

## Sources

* [Ansible role documentation](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html)
* [Ansible Molecule documentation](https://molecule.readthedocs.io/)
