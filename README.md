# aspects_haproxy
Install and configure HAProxy.

> Remember that blank lines and indentation do not matter to HAProxy.

# Requirements
The `aspects_packages` role should be installed and enabled.

If you do not use `aspects_packages`, then you need to install HAProxy another way.

## aspects_packages
This role can use aspects_packages to install the HAProxy repository and packages. 

`aspects_packages_add_repo_apt_repos` and `aspects_packages_packages` are configured with the 2.2 LTS release repo and
package in [defaults/main.yml](defaults/main.yml).

## `hash_behaviour=merge`
Ansible officially does not like `hash_behaviour=merge`. You can use this role without it.

But if you take care in how you structure your project, `hash_behaviour=merge` can save a lot of time. So I encourage
you to take a good look at using it.

# Role Variables
## aspects_haproxy_enabled
Boolean. False to skip role tasks. True to run role tasks.

## aspects_haproxy_config_global
The `global` section of the `/etc/haproxy/haproxy.cfg` file.

The value in [defaults/main.yml](defaults/main.yml) is from the Ubuntu 20.04 package.

[The Official Documentation](http://cbonte.github.io/haproxy-dconv/2.2/configuration.html#3)

## aspects_haproxy_config_defaults
The `defaults` section of the `/etc/haproxy/haproxy.cfg` file.

The value in [defaults/main.yml](defaults/main.yml) is from the Ubuntu 20.04 package.

See this section of the [official documentation](http://cbonte.github.io/haproxy-dconv/2.2/configuration.html#4) for a description of what this section does.

## aspects_haproxy_before
Configuration in this section will go directly after the defaults section, and before any other sections.

```yaml
aspects_haproxy_before:
  01234something:
    enabled: < True | False >
    block: |
      # config goes here.
```

## aspects_haproxy_userlists
As far as I can tell, the best place for userlists is before any frontend or backend configuration. So
this config section is placed before the frontend section.

```yaml
aspects_haproxy_userlists:
  01234something:
    enabled: < True | False >
    block: |
      # config goes here.
      userlist < name of list >
      < lines like: user alice insecure-password passchangeme >
```

## aspects_haproxy_frontends
A dictionary of `frontend` configurations. 

The dictionary is sorted by key on both the top level frontend section, and the lower level 'sections' section.

```yaml
aspects_haproxy_frontends:
  < some useful name for the frontend >:
    enabled: < True | False >
    mode: < Whichever value of the mode you want >
    options: |
      < a block of text 
        containing the various haproxy options
        you wish to use. >
    binds: |
      < a block of text
        containing the bind configuration you
        wish to use >
    sections:
      < section key, start with a number to make sorting clear >:
        enabled: < True | False >
        block: |
          < a block HAProxy configuration for the frontend >
      ...
      < another section key, start with a number to make sorting clear >:
        enabled: < True | False >
        block: |
          < a block HAProxy configuration for the frontend >
```

## aspects_haproxy_backends
A dictionary of `backend` configurations. 

The dictionary is sorted by key.

```yaml
aspects_haproxy_backends:
  < a useful key for the backend, start with a number to make sorting clear >:
    enabled: < True | False >
    block: |
      < a block of HAProxy backend configuration >
  ...
  < another useful key for the backend, start with a number to make sorting clear >:
    enabled: < True | False >
    block: |
      < a block of HAProxy backend configuration >
``` 
## aspects_haproxy_listens
A dictionary of `listen configurations. 

The dictionary is sorted by key.

```yaml
aspects_haproxy_listens:
  < a useful key for the listen, start with a number to make sorting clear >:
    enabled: < True | False >
    mode: < Whichever value of the mode you want >
    options: |
      < a block of text 
        containing the various haproxy options
        you wish to use. >
    binds: |
      < a block of text
        containing the bind configuration you
        wish to use >
  ...
  < another useful key for the listen, start with a number to make sorting clear >:
    enabled: < True | False >
    mode: < Whichever value of the mode you want >
    options: |
      < a block of text 
        containing the various haproxy options
        you wish to use. >
    binds: |
      < a block of text
        containing the bind configuration you
        wish to use >
```

## aspects_haproxy_after
Configuration in this section will go after all other sections.

```yaml
aspects_haproxy_after:
  01234something:
    enabled: < True | False >
    block: |
      # config goes here.
```

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
    aspects_haproxy_frontends:
      http_https:
        enabled: True
        mode: "http"
        options: |
          option	httplog
          option forwardfor
        binds: |
          bind *:80
          bind *:443 ssl crt /etc/haproxy/certs/
        sections:
          99999999default_backend:
            enabled: True
            block: |
              # Default backend
              default_backend back_base
          00000001capture_well_known:
            enabled: True
            block: |
              # capture .well-known request
              acl acl_path_dot_wellknown path_beg /.well-known/acme-challenge/
          00000002site_acls:
            enabled: True
            block: |
              # mailserver acl's
              acl acl_host_mailserver_domain hdr_dom(host) -i webmail.mailserver.com
              acl acl_host_mailserver_domain hdr_dom(host) -i account.mailserver.com
              acl acl_host_mailserver_domain hdr_dom(host) -i spamfilter.mailserver.com
              acl acl_host_mailserver_domain hdr_dom(host) -i smtp.mailserver.com
              acl acl_host_mailserver_domain hdr_dom(host) -i imap.mailserver.com
              acl acl_host_mailserver_domain hdr_dom(host) -i pop.mailserver.com
              acl acl_host_mailserver_domain hdr_dom(host) -i pop3.mailserver.com
              acl acl_host_webmail_mailserver_com hdr_dom(host) -i webmail.mailserver.com
              acl acl_host_account_mailserver_com hdr_dom(host) -i account.mailserver.com
              acl acl_host_spamfilter_mailserver_com hdr_dom(host) -i spamfilter.mailserver.com
          00000003http_to_https:
            enabled: True
            block: |
              # redirect to https
              http-request redirect scheme https if !{ ssl_fc } !acl_path_dot_wellknown
          00000004use:
            enabled: True
            block: |
              # mail_acme
              use_backend back_mailserver_acme if acl_path_dot_wellknown acl_host_mailserver_domain
              # raygun_mail_webmail
              use_backend back_mailserver_webmail if acl_host_webmail_mailserver_com
              # raygun_mail_vimbadmin
              use_backend back_mailserver_vimbadmin if acl_host_account_mailserver_com
              # raygun_mail_spamfilter
              use_backend back_mailserver_spamfilter if acl_host_spamfilter_mailserver_com
    aspects_haproxy_backends:
      99999999back_base:
        enabled: True
        block: |
          backend back_base
          mode	http
          server base1 127.0.1.2:59999 check port 59999
      00000001back_mailserver_acme:
        enabled: True
        block: |
          backend back_mailserver_acme
          mode	http
          server raygun_mail_acme1 127.0.10.254:1099
      00000002back_mailserver_webmail:
        enabled: True
        block: |
          backend back_mailserver_webmail
          mode	http
          server webmail1 127.0.11.8:1015
      00000003back_mailserver_vimbadmin:
        enabled: True
        block: |
          backend back_mailserver_vimbadmin
          mode	http
          server vimbadmin1 127.0.10.4:1009
      00000004back_mailserver_spamfilter:
        enabled: True
        block: |
          backend back_mailserver_spamfilter
          mode	http
          server spamfilter1 127.0.10.6:1013
    aspects_haproxy_listens:
      00000001mailserver_imaps:
        enabled: True
        mode: "tcp"
        binds: |
          bind *:993
        block: |
          server imaps1 127.0.10.2:1000 check port 1000 send-proxy
```

# License
MIT
