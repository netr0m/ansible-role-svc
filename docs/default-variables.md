# Default Variables
## All

### Domain name

```yaml
svc_domain: local
```
### Environment

```yaml
svc_env: "{{ lookup('env', 'ENVIRONMENT') | default('production') }}"
```
### Username of the user owning the files

```yaml
svc_user_name: 'butler'
```
### Group name of the group that should own the files

```yaml
svc_group_name: 'butlers'
```
### Optionally provide the UID of the user. If absent, the UID will be looked up

```yaml
svc_user_uid: undefined
```
### Optionally provide the GID of the group. If absent, the GID will be looked up

```yaml
svc_group_gid: undefined
```
### Timezone

```yaml
svc_tz: Etc/UTC
```
## Directories

### Manage directories

```yaml
svc_manage_directories: true
```
### Directory to store service data

```yaml
svc_directory:
  path: '/opt/svc'
  # Default permissions
  owner: "{{ svc_user_name }}"
  group: "{{ svc_group_name }}"
  mode: 740
```
### Subdirectories

```yaml
svc_subdirectories:
  cfg:
    path: "{{ svc_directory.path }}/cfg"
  log:
    path: "{{ svc_directory.path }}/log"
  data:
    path: "{{ svc_directory.path }}/data"
```
## Packages

### Manage packages

```yaml
svc_manage_packages: true
```
### Packages to install. See vars/<ansible_os_family>.yml

```yaml
svc_packages: []
```
### Pip packages to install

```yaml
svc_packages_pip:
  - 'docker'
```
## Services

### Default restart policy

```yaml
svc_restart_policy: 'always'
```
### Whether to force pull container images

```yaml
svc_force_pull: false
```
### Default logging driver for Docker containers

```yaml
svc_log_driver: local
```
### Default logging options - see https://docs.docker.com/config/containers/logging/configure/

```yaml
svc_log_options:
  max-size: 20m
  max-file: '5'
  compress: 'true'
```
### Manage docker networks

```yaml
svc_manage_docker_networks: true
```
### Configure Traefik

```yaml
svc_use_traefik: true
```
## Traefik

### Directories for Traefik

```yaml
svc_traefik_directories:
  cfg:
    path: "{{ svc_subdirectories.cfg.path }}/traefik"
  log:
    path: "{{ svc_subdirectories.log.path }}/traefik"
```
### Log level

```yaml
svc_traefik_log_level: 'INFO'
```
### Whether to enable the Traefik dashboard

```yaml
svc_traefik_enable_dashboard: true
```
### Whether to enable DEBUG mode

```yaml
svc_traefik_debug: false
```
### Whether to allow insecure access

```yaml
svc_traefik_insecure: false
```
### Whether to expose Docker containers by default

```yaml
svc_traefik_exposed_by_default: false
```
### Whether to automatically retrieve TLS certificates. Requires 'svc_traefik_dns_challenge_provider' and 'svc_traefik_acme_settings'.

```yaml
svc_traefik_automatic_https: true
```
### Challenge provider to use for automatic TLS certificate acquisition. See https://doc.traefik.io/traefik/https/acme/providers

```yaml
svc_traefik_dns_challenge_provider: 'cloudflare'
```
### Whether to use the staging servers (recommended for testing)

```yaml
svc_traefik_letsencrypt_staging: false
```
### Environment variables for Traefik to automatically acquire TLS certificates

```yaml
svc_traefik_acme_settings:
  CF_DNS_API_TOKEN: "{{ lookup('env', 'CF_DNS_API_TOKEN') | default('undefined') }}"
```
### traefik container hostname

```yaml
svc_traefik_container_hostname: traefik
```
### traefik version

```yaml
svc_traefik_version: latest
```
### traefik container image

```yaml
svc_traefik_container_image: "traefik:{{ svc_traefik_version }}"
```
### traefik container memory

```yaml
svc_traefik_container_memory: 1g
```
### traefik container ports

```yaml
svc_traefik_container_ports:
  http: 80
  https: 443
```
### Used as the 'average' parameter for the rate limiting middleware

```yaml
svc_traefik_middleware_rate_limit_average: 50
```
### Used as the 'burst' parameter for the rate limiting middleware

```yaml
svc_traefik_middleware_rate_limit_burst: 100
```
### Extra hosts for Traefik. See templates/etc/traefik/config/http.yml

```yaml
svc_traefik_extra_hosts: []
  # - name: example
  #   shortname: ex
  #   middlewares: []
  #   protocol: https
  #   ip_addr: 10.10.10.10
  #   port: 8080
```
### Extra middlewares for Traefik. See templates/etc/traefik/config/http.yml

```yaml
svc_traefik_middlewares: {}
  # example-mwr:
  #   headers:
  #     customRequestHeaders:
  #       Authorization: ''
  #       X-Forwarded-Proto: 'https'
```
### Extra certificates for Traefik. See templates/etc/traefik/traefik.yml
First entry in the list will be used as the default, if any

```yaml
svc_traefik_certificates: []
  # - crt: /etc/traefik/tls/domain.tld.crt
  #   key: /etc/traefik/tls/domain.tld.key
```
## Docker Socket Proxy

### socketproxy container hostname

```yaml
svc_socketproxy_container_hostname: "socket-proxy"
```
### socketproxy version

```yaml
svc_socketproxy_version: "0.1.1"
```
### socketproxy container image

```yaml
svc_socketproxy_container_image: "ghcr.io/tecnativa/docker-socket-proxy:{{ svc_socketproxy_version }}"
```
### socketproxy container memory

```yaml
svc_socketproxy_container_memory: 1g
```
### socketproxy container restart policy

```yaml
svc_socketproxy_restart_policy: "unless-stopped"
```
### Settings for the docker socket proxy. See https://github.com/Tecnativa/docker-socket-proxygrant-or-revoke-access-to-certain-api-sections

```yaml
socket_proxy_settings:
  LOG_LEVEL: "info"
  EVENTS: "1"
  PING: "1"
  VERSION: "1"
  CONTAINERS: "1"
  IMAGES: "0"
  NETWORKS: "0"
  VOLUMES: "0"
  POST: "0"
  SERVICES: "0"
  INFO: "0"
  TASKS: "0"
  AUTH: "0"
  SECRETS: "0"
  BUILD: "0"
  COMMIT: "0"
  CONFIGS: "0"
  DISTRIBUTION: "0"
  EXEC: "0"
  GRPC: "0"
  NODES: "0"
  PLUGINS: "0"
  SESSION: "0"
  SWARM: "0"
  SYSTEM: "0"
```