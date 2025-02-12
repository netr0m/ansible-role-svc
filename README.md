# Ansible Role: Service

[![CI](https://github.com/netr0m/ansible-role-svc/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/netr0m/ansible-role-svc/actions/workflows/ci.yml)

An Ansible role for shared configurations for service hosts. Handles common tasks for hosts used to deploy services, including configuring Traefik for reverse proxying (with [`docker-socket-proxy`](https://github.com/Tecnativa/docker-socket-proxy)), creating directories for configuration files and data, as well as defining some defaults for other containers deployed through e.g. [`netr0m.infra`](https://github.com/netr0m/ansible-role-infra).

## Installation

```sh
$ ansible-galaxy install git+https://github.com/netr0m/ansible-role-svc.git
```

## Requirements

None

## Role Variables

Available variables are listed in [`docs/default-variables.md`](./docs/default-variables.md) (see [`defaults/main.yml`](./defaults/main.yml))

### Minimal configuration [required]

Most of the defaults variables can be used as-is, but there are a few variables that must be set:

```yml
# Username of the user owning the files
svc_user_name: 'service_username'
# Group name of the group that should own the files
svc_group_name: 'service_groupname'
```

### Recommended configuration changes

```yml
# Domain name
svc_domain: mydomain.tld
```

#### Automated HTTPS certificate acquisition (with Cloudflare as an example)

Uses the DNS challenge method, allowing internal (non-publicly accessible hosts) to acquire certificates. *All certificates are wildcard-certificates, to avoid exposing internal hostnames/services used.*

```yml
# Domain name
svc_domain: mydomain.tld

# Challenge provider to use for automatic TLS certificate acquisition. See https://doc.traefik.io/traefik/https/acme/#providers
svc_traefik_dns_challenge_provider: 'cloudflare'

# Environment variables for Traefik to automatically acquire TLS certificates
svc_traefik_acme_settings:
  # API token with DNS:Edit permission
  CF_DNS_API_TOKEN: "{{ lookup('env', 'CF_DNS_API_TOKEN') | default('undefined') }}"
```

#### Extending Traefik

##### Adding additional middlewares
*See default middlewares in [vars/main.yml](vars/main.yml) under `svc_traefik_default_middlewares`.*

See the [Traefik Docs on HTTP Middlewares](https://doc.traefik.io/traefik/middlewares/http/overview/#available-http-middlewares) for details.

```yml
svc_traefik_extra_middlewares:
  my-custom-mwr:
    headers:
      customRequestHeaders:
        Authorization: ''
        X-Forwarded-Proto: 'https'
    addPrefix:
      prefix: "/api"
```

##### Adding additional entryPoints
*See default entryPoints in [traefik.yml.j2](templates/etc/traefik/traefik.yml.j2)*

```yml
svc_traefik_extra_entrypoints:
  - name: dns
    port: 53
  - name: dnsUdp
    port: 53/udp
```

##### Adding additional hosts without using Docker container labels (also applies to non-Docker hosts and services on remote hosts)
```yml
svc_traefik_extra_hosts:
  - name: example01
    shortname: ex01
    middlewares: []
    protocol: https
    ip_addr: 10.10.10.10
    port: 8080
  - name: jellyfin
    subdomain: media
    shortname: jlf
    middlewares:
      - rate-limit-mwr
      - lan-mwr
      - my-custom-mwr
    protocol: https
    ip_addr: 10.10.10.11
    port: 8081
```

The attribute '`shortname`' is used as the identifier for the Traefik `Router` and `Service` components (e.g. `jlf-rtr` and `jlf-svc`). If the attribute '`subdomain`' is absent, the '`shortname`' attribute will be used as the subdomain as well (e.g. `ex01.<svc_domain>`).

##### Additional Traefik config YAML files

You may also add any valid Traefik config file under `"{{ svc_traefik_directories.cfg.path }}"` (defaults to `/opt/svc/cfg/traefik/config`)

## Dependencies

See [ansible-requirements.yml](./ansible-requirements.yml) for a list

### Installation
```sh
ansible-galaxy collection install -r ansible-requirements.yml
ansible-galaxy role install -r ansible-requirements.yml
```

## Example Playbook

```yml
---
- name: Example Playbook
  hosts: all
  become: true
  gather facts: true

  roles:
    - { role: netr0m.svc }
...

```

## Development
This project uses [pre-commit](https://pre-commit.com/).

Currently, there are three hooks:
- [ansible-lint](https://pypi.org/project/ansible-lint/)
- [yamllint](https://pypi.org/project/yamllint/)
- [encryption-check](./scripts/encryption-check.sh)

To run `pre-commit` manually, run `pre-commit run -a`

### Requirements
To run pre-commit, you need three things:
1. A virtual environment in the parent directory of this repository
  - `$ python3 -m venv ../.venv`
  - `$ source ../.venv/bin/activate`
2. The Python dependencies (see [requirements.txt](./requirements.txt))
  - `$ pip install -r requirements.txt`
3. Pre-commit hooks installed
  - `$ pre-commit install`

### Updating the 'variables' docs
This project provides a script for generating markdown files representing ansible (YAML) variable definitions.

An example can be seen in [`docs/default-variables.md`](./docs/default-variables.md), which is generated from the variables defined in [`defaults/main.yml`](./defaults/main.yml).

#### Running the script
To run the generator, issue the following command. If no parameters are specified, this will generate a markdown file based on the variables in `defaults/main.yml`, and write it to `docs/default-variables.md`.

```sh
$ python3 generate-vars-md.py

# Display help message
$ python3 generate-vars-md.py --help

# Specify alternative input and output paths
$ python3 generate-vars-md.py --in-file vars/debian.yml --out-file docs/debian-vars.md --title "Debian Variables"
```

## License

[MIT](./LICENSE)

## Author Information

This role was created in 2022 by [netr0m](https://github.com/netr0m)
