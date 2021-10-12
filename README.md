# Ansible role Caddy

This role installs a plain caddy server on a Debian-based system and can, if you want, also place a basic reverse proxy configuration.

## Variables

If you only want to install Caddy, you don't need to set any variables. If you want to configure Caddy as a reverse proxy as well, you can provide an array of objects named `caddy_sites` with the following values:

* `additional_forwarding_ports`: Allows to define a list with additional ports where Caddy should listen for this domain and forward to HTTPS.
* `allowlist`: An array if IP addresses in CIDR-notation which are allowed to access this site (Optional). All other visitors receive a 404 error.
* `certificate_file`: You can set this variable if you want to provide the certificate by yourself (Optional). The certificate needs permissions `0640`, with root as Owner and Caddy as Group.
* `certificate_key`: You can set this variable if you want to provide the certificate by yourself (Optional).
* `domain`: The domain caddy should listen to.

Afterwards, you can define a list of `routes` composing of the following values:

* `path`: Path that should be matched. Let it empty for everything or e.g. `/api/*` for something specific.
* `reverse_proxy_destination`: Where the requested should be proxied.
* `strip_prefix`: If set, the matched `path` will be removed from the request to the destination system. This means, if somebody requests the route `/api/v1/hello` at the reverse proxy and you set `/api/*` as path, the request will be sent as `/v1/hello` to the destination system.

Certificates, domain etc. are always defined for one site and cannot be redefined for a route.

## Sample playbooks

Basic installation:

```yaml
---
- hosts: "reverse_proxy_servers"
  become: yes

  roles:
    - role: caddy
      become: true
```

With reverse proxy configuration:

```yaml
---
- hosts: "reverse_proxy_servers"
  become: yes

  tasks:
    - name: Write out certificates
      copy:
        content: "{{ vault_simpli_self_signed_certificate }}"
        dest: "/etc/caddy/simpli.cert.pem"
        group: caddy
        owner: root
        mode: 0640
      notify:
        - Restart Caddy

    - name: Write out certificate keys
      copy:
        content: "{{ vault_simpli_self_signed_key }}"
        dest: "/etc/caddy/simpli.key.pem"
        group: caddy
        owner: root
        mode: 0640
      notify:
        - Restart Caddy

  roles:
    - role: caddy
      become: true
```
