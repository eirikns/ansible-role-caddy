# ansible-role-caddy

An Ansible role that installs and configures [Caddy](https://caddyserver.com) on Debian and Ubuntu. Caddy is a web server with automatic HTTPS via Let's Encrypt — no separate certbot role required.

## Requirements

- Target hosts running Debian or Ubuntu
- Ports 80 and 443 reachable from the internet for automatic TLS (or use a DNS challenge)

Caddy is installed from the distribution's standard package repositories. The version available may lag behind the latest upstream release.

## Role Variables

See [`defaults/main.yaml`](defaults/main.yaml) for the full list of variables and their defaults. The most important ones are described below.

```yaml
caddy_acme_email: "admin@example.com"
```

Email address registered with Let's Encrypt for certificate notifications. Optional but recommended.

```yaml
caddy_acme_ca: "https://acme-staging-v02.api.letsencrypt.org/directory"
```

Override the ACME CA. Useful during testing to avoid Let's Encrypt production rate limits. Leave empty to use Let's Encrypt production.

```yaml
caddy_vhosts:
  - domain: "example.com"
    reverse_proxy: "localhost:8080"
    dial_timeout: "10s"
    response_header_timeout: "120s"
    encode: true
```

List of virtual hosts to configure. Each entry supports:

- `domain` — the domain name Caddy listens on (e.g. `"example.com"`)
- `tls_email` — email for TLS certificate registration; overrides `caddy_acme_email` for this vhost. When either this or `caddy_acme_email` is set, a `tls` directive is added to the site block to enable ACME certificate provisioning.
- `reverse_proxy` — upstream address to proxy requests to (e.g. `"localhost:8080"`)
- `dial_timeout` — timeout for connecting to the upstream (e.g. `"10s"`)
- `response_header_timeout` — timeout waiting for the upstream to send response headers (e.g. `"120s"`). Increase this for slow backends.
- `encode` — enable gzip and zstd response compression (default: `false`)
- `root` — document root; enables `file_server` when set
- `extra_config` — raw Caddyfile directives appended inside the vhost block

```yaml
caddy_config: |
  example.com {
      reverse_proxy localhost:8080
  }
```

Raw Caddyfile content. When set, this is written directly to `/etc/caddy/Caddyfile` and `caddy_vhosts` is ignored. Use this for configurations that go beyond what `caddy_vhosts` supports.

## Example Playbook

Reverse proxy with automatic HTTPS:

```yaml
- hosts: servers
  become: true
  roles:
    - role: eirikns.caddy
      vars:
        caddy_acme_email: "admin@example.com"
        caddy_vhosts:
          - domain: "jira.example.com"
            reverse_proxy: "localhost:8080"
            response_header_timeout: "120s"
            encode: true
```

## License

MIT No Attribution

## Author

This Ansible role was created in 2026 by Eirik Nicolai Synnes and published as an Open Source project.

## Acknowledgements

This role uses Docker images created by Jeff Geerling for testing. The approach for developing this role, as well how to use GitHub Actions to manage it, is inspired by his work. You can find him on GitHub at https://github.com/geerlingguy.
