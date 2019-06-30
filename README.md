#Haproxy installer and configurator Ansible role
---
This role is meant to install haproxy in Debian for the moment, 
and also configure it the more coherent way i have in mind.

For the moment is a basic functionality release enough for a multi
frontend/backend easy config. Anyway, the idea is to allow advanced
config the easiest and more documented way.

Feel free to contritube with your ideas or improve repo with fixes, tests, etc.

###playbook.yml example
```yaml
- hosts: test
  roles:
    - haproxy
```
exec only config:
````yaml
ansible-playbook /ansible/playbook.yml --tags haproxy_configuration
````

###host_vars example

```yaml
haproxy_frontend:
  - name: www-http
    description: HTTPS config
    bind: "*:80"
    reqadd: "X-Forwarded-Proto:\ http"
    default_backend: fallback_backend

  - name: www-https
    description: HTTPS config
    bind: "*:443"
    reqadd: "X-Forwarded-Proto:\ http"
    acls:
      - name: host_name
        condition: "hdr(host) -i www.domain.com"
        backend: "domain_backend"
    ssl_paths:
      - /etc/haproxy/certs/certname.tld.pem
      - /etc/haproxy/certs/subdomain.certname.tld.pem
      - /etc/haproxy/certs/algun.certname.pem

haproxy_backend:
  - name: fallback_backend
    servers:
      - name: blind
        bind: 127.0.0.1:80
  - name: domain_backend
    params: check
    redirect: scheme https if !{ ssl_fc }
    servers:
      - name: www-01
        bind: 0.0.0.0:80
  - name: domain_2
    params: check
    servers:
      - name: www-01
        bind: 1.1.1.1:111
        params: check
      - name: www-02
        bind: 1.1.1.2:222
        params: check
```

Why this repo?
---
I've made it because after find some roles to do this job, the host_vars configuration there
was quite repetitive and confusing in some points.

Also because i expected a properly formatted haproxy.cfg file, and found repos
was not taking care about readability of sources from terminal.

Was also needed to edit in several places some things that should be calculated automatically.
