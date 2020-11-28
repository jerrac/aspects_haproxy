# aspects_haproxy
Install and configure HAProxy.

# Requirements
The `aspects_packages` role should be installed and enabled.

If you do not use `aspects_packages`, then you need to install HAProxy another way.

## aspects_packages
This role can use aspects_packages to install the HAProxy repository and packages. 

`aspects_packages_add_repo_apt_repos` and `aspects_packages_packages` are configured with the 2.2 LTS release repo and
package in [defaults/main.yml](defaults/main.yml).

# Role Variables
## aspects_haproxy_enabled
Boolean. False to skip role tasks. True to run role tasks.

## aspects_haproxy_config_global
The `global` section of the `/etc/haproxy/haproxy.cfg` file.

The value in [defaults/main.yml](defaults/main.yml) is from the Ubuntu 20.04 package.

## aspects_haproxy_config_defaults
The `defaults` section of the `/etc/haproxy/haproxy.cfg` file.

The value in [defaults/main.yml](defaults/main.yml) is from the Ubuntu 20.04 package.

# Dependencies
* aspects_packages (optional)

# Example Playbook

```yaml
- hosts: servers
  roles:
     - aspects_haproxy
  vars:
    aspects_packages_enabled: True
    aspects_haproxy_enabled: True
    aspects_haproxy_config:
      test:
        enabled: True
        block: |
          frontend http-in
              bind *:80
              default_backend servers
          backend servers
              server server1 127.0.0.1:8000 maxconn 32
```

# License
MIT
