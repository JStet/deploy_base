# deploy_base


This role creates a foundation for the deployment of dockerized web apps by setting up a linux server (including security) and deploying a docker swarm containing monitoring services, an alert manager and a reverse proxy.

Also find it on [Ansible Galaxy](https://galaxy.ansible.com/jstet/deploy_base)

One use of this setup is to allow you to deploy multiple apps on one server. This can help you save hosting costs. Traefik will redirect incoming requests to the right docker container. Find an instruction on how to deploy an app to this setup and how to add an automatic deploy function with Github Workflows in [this repo](https://github.com/jstet/jstet_iac).

This docker swarm contains:

- Traefik
- Prometheus
- CAdvisor
- Node Exporter
- Alertmanager
- Grafana
- Plausible Web Analytics
- Local docker registry

## Requirements

A user with posswordless sudo privileges should be set up on the server. Personally I take care of that with a cloud config file. You need to "become" and gather facts for this role to work.

## Role Variables

```
ansible-galaxy install jstet.deploy_base
```

## Role Variables


A user with passwordless sudo privileges that will execute all docker tasks.
```
user: user
```

Extra packages you want to install on the server.
```
packages:
  - htop
  - vim
  - net-tools
```

Vars needed for setting up Grafana. The password for the user "admin" and the domain on which you want to reach Grafana.
```
GRAFANA_PW: 1234
MONITORING_DOMAIN: monitoring.localhost
```

The email used for administering a certificate with letsencrypt.
```
LETSENCRYPT_EMAIL: mail@example.com
```

## Dependencies


This role uses jstet.initial_server_setup and geerlingguy.docker. Will be installed automatically with the needed parameters.


## Example Playbook
---
```
- hosts: all
  gather_facts: yes
  become: no
  vars_files:
    - vars/vault.yml
  roles:
    - role: deploy_base
      vars:
          packages: 
              - htop
              - vim
              - net-tools
          user: user
          GRAFANA_PW: "{{ GRAFANA_PW_VAULT }}"
          LETSENCRYPT_EMAIL: example@mail,de
          MONITORING_DOMAIN: data.example.net
          TRAEFIK_USER: admin
  tasks:
```

## License


MIT

## Author Information

jstet.net

## Sources

### iptables and docker
- https://github.com/chaifeng/ufw-docker#solving-ufw-and-docker-issues
### Prometheus
#### Node exporter
- https://prometheus.io/docs/guides/node-exporter/
### Docker and Prometheus
- https://schroederdennis.de/allgemein/prometheus-mit-docker-installieren-und-einrichten-anleitung/
- https://github.com/vegasbrianc/prometheus/blob/master/docker-compose.yml
- https://prometheus.io/docs/guides/cadvisor/
- https://logz.io/blog/prometheus-tutorial-docker-monitoring/#systemdocker
### Grafana
- https://www.howtogeek.com/devops/how-to-run-grafana-in-a-docker-container/
### nginx-exporter
- https://github.com/nginxinc/nginx-prometheus-exporter/issues/30
### promtail and loki
- https://blog.lrvt.de/log-visualization-with-grafana-loki-promtail/
### Traefik
- https://goneuland.de/traefik-v2-reverse-proxy-fuer-docker-unter-debian-10-einrichten/
### Docker Swarm
- https://gabrieltanner.org/blog/docker-swarm/
- https://dockerswarm.rocks/
- https://github.com/vegasbrianc/docker-traefik-prometheus
### Alertmanager
- https://github.com/vegasbrianc/prometheus
- https://medium.com/techno101/how-to-send-a-mail-using-prometheus-alertmanager-7e880a3676db
- https://api.slack.com/messaging/webhooks
- https://prometheus.io/docs/alerting/latest/configuration/#configuration-file
- https://grafana.com/blog/2020/02/25/step-by-step-guide-to-setting-up-prometheus-alertmanager-with-slack-pagerduty-and-gmail/

### Plausible
- https://plausible.io/docs/self-hosting

## Misc


### Testing alertmanager

```
docker service create --replicas 1 --name test --network=monitoring alpine/curl:3.14
docker exec -it .... sh
curl -H 'Content-Type: application/json' -d '[{"labels":{"alertname":"hallo hallo"}}]' http://alertmanager:9093/api/v1/alerts
```
