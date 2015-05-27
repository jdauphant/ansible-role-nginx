nginx
=====

This role installs and configures the nginx web server. The user can specify
any http configuration parameters they wish to apply their site. Any number of
sites can be added with configurations of your choice.

Requirements
------------

This role requires Ansible 1.4 or higher and platform requirements are listed
in the metadata file.

You need EPEL to be setup before running this role.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows.

```yaml
# The user to run nginx
nginx_user: "www-data"

# A list of directives for the events section.
nginx_events_params:
 - worker_connections 512
 - debug_connection 127.0.0.1
 - use epoll
 - multi_accept on

# A list of hashs that define the servers for nginx,
# as with http parameters. Any valid server parameters
# can be defined here.
nginx_sites:
 default:
     - listen 80
     - server_name _
     - root "/usr/share/nginx/html"
     - index index.html
 foo:
     - listen 8080
     - server_name localhost
     - root "/tmp/site1"
     - location / { try_files $uri $uri/ /index.html; }
     - location /images/ { try_files $uri $uri/ /index.html; }
 bar:
     - listen 9090
     - server_name ansible
     - root "/tmp/site2"
     - location / { try_files $uri $uri/ /index.html; }
     - location /images/ {
         try_files $uri $uri/ /index.html;
         allow 127.0.0.1;
         deny all;
       }

# A list of hashs that define additional configuration
nginx_configs:
  proxy:
      - proxy_set_header X-Real-IP  $remote_addr
      - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
  upstream:
      - upstream foo { server 127.0.0.1:8080 weight=10; }
  geo:
      - geo $local {
          default 0;
          127.0.0.1 1;
        }
  gzip:
      - gzip on
      - gzip_disable msie6

# A list of hashs that define user/password files
nginx_auth_basic_files:
   demo:
     - foo:$apr1$mEJqnFmy$zioG2q1iDWvRxbHuNepIh0 # foo:demo , generated by : htpasswd -nb foo demo
     - bar:$apr1$H2GihkSo$PwBeV8cVWFFQlnAJtvVCQ. # bar:demo , generated by : htpasswd -nb bar demo

```

Examples
========

1) Install nginx with HTTP directives of choices, but with no sites
configured and no additionnal configuration:

```yaml
- hosts: all
  roles:
  - {role: nginx,
     nginx_http_params: ["sendfile on", "access_log /var/log/nginx/access.log"]
                          }
```

2) Install nginx with different HTTP directives than previous example, but no
sites configured and no additionnal configuration.

```yaml
- hosts: all
  roles:
  - {role: nginx,
     nginx_http_params: ["tcp_nodelay on", "error_log /var/log/nginx/error.log"]}
```

Note: Please make sure the HTTP directives passed are valid, as this role
won't check for the validity of the directives. See the nginx documentation
for details.

3) Install nginx and add a site to the configuration.

```yaml
- hosts: all

  roles:
  - role: nginx
    nginx_http_params:
      - sendfile "on"
      - access_log "/var/log/nginx/access.log"
    nginx_sites:
      bar:
        - listen 8080
        - location / { try_files $uri $uri/ /index.html; }
        - location /images/ { try_files $uri $uri/ /index.html; }
    nginx_configs:
      proxy:
        - proxy_set_header X-Real-IP  $remote_addr
        - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
```

Note: Each site added is represented by list of hashes, and the configurations
generated are populated in /etc/nginx/site-available/, a link is from /etc/nginx/site-enable/ to /etc/nginx/site-available

The file name for the specific site configurtaion is specified in the hash
with the key "file_name", any valid server directives can be added to hash.
Additional configuration are created in /etc/nginx/conf.d/

4) Install Nginx , add 2 sites (different method) and add additional configuration

```yaml
---
- hosts: all
  roles:
    - role: nginx
      nginx_http_params:
        - sendfile on
        - access_log /var/log/nginx/access.log
      nginx_sites:
         foo:
           - listen 8080
           - server_name localhost
           - root /tmp/site1
           - location / { try_files $uri $uri/ /index.html; }
           - location /images/ { try_files $uri $uri/ /index.html; }
         bar:
           - listen 9090
           - server_name ansible
           - root /tmp/site2
           - location / { try_files $uri $uri/ /index.html; }
           - location /images/ { try_files $uri $uri/ /index.html; }
      nginx_configs:
         proxy:
            - proxy_set_header X-Real-IP  $remote_addr
            - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
```

5) Install Nginx , add 2 sites, add additional configuration and an upstream configuration block

```yaml
---
- hosts: all
  roles:
    - role: nginx
      nginx_http_params:
        - sendfile on
        - access_log /var/log/nginx/access.log
      nginx_sites:
        foo:
           - listen 8080
           - server_name localhost
           - root /tmp/site1
           - location / { try_files $uri $uri/ /index.html; }
           - location /images/ { try_files $uri $uri/ /index.html; }
        bar:
           - listen 9090
           - server_name ansible
           - root /tmp/site2
           - if ( $host = example.com ) { rewrite ^(.*)$ http://www.example.com$1 permanent; }
           - location / { try_files $uri $uri/ /index.html; }
           - location /images/ { try_files $uri $uri/ /index.html; }
           - auth_basic            "Restricted"
           - auth_basic_user_file  auth_basic/demo
      nginx_configs:
        proxy:
            - proxy_set_header X-Real-IP  $remote_addr
            - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
        upstream:
            # Results in:
            # upstream foo_backend {
            #   server 127.0.0.1:8080 weight=10;
            # }
            - upstream foo_backend { server 127.0.0.1:8080 weight=10; }
      nginx_auth_basic_files:
        demo:
           - foo:$apr1$mEJqnFmy$zioG2q1iDWvRxbHuNepIh0 # foo:demo , generated by : htpasswd -nb foo demo
           - bar:$apr1$H2GihkSo$PwBeV8cVWFFQlnAJtvVCQ. # bar:demo , generated by : htpasswd -nb bar demo
```

6) Example to use this role with my ssl-certs role to generate or copie ssl certificate ( https://galaxy.ansible.com/list#/roles/3115 )
```yaml
 - hosts: all
   roles: 
     - jdauphant.ssl-certs
     - role: jdauphant.nginx
       nginx_configs: 
          ssl:
               - ssl_certificate_key {{ssl_certs_privkey_path}}
               - ssl_certificate     {{ssl_certs_cert_path}}
       nginx_sites:
          default:
               - listen 443 ssl
               - server_name _
               - root "/usr/share/nginx/html"
               - index index.html
```

Dependencies
------------

None

License
-------
BSD

Author Information
------------------

- Original : Benno Joy
- Modified by : DAUPHANT Julien

