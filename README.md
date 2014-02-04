nginx
=====

This role installs and configures the nginx web server. The user can specify
any http configuration parameters they wish to apply their site. Any number of
sites can be added with configurations of your choice.

Requirements
------------

This role requires Ansible 1.4 or higher and platform requirements are listed
in the metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows.

    # The max clients allowed
    nginx_max_clients: 512 

    # The user to run nginx
    nginx_user: "www-data"

    # A list of the http paramters. Note that any
    # valid nginx http paramters can be added here.
    # (see the nginx documentation for details.)
    nginx_http_params:                                    
      - sendfile: "on"                                      
      - tcp_nopush: "on"
      - tcp_nodelay: "on"
      - keepalive_timeout: "65"
      - access_log: "/var/log/nginx/access.log"
      - error_log: "/var/log/nginx/error.log"

    # A list of hashs that define the servers for nginx,
    # as with http parameters. Any valid server parameters
    # can be defined here.
    nginx_sites:                                         
       - server:
          file_name: foo
          vars:
           - listen 8080
           - server_name localhost
           - root "/tmp/site1"
           - location / { try_files $uri $uri/ /index.html; }
           - location /images/ { try_files $uri $uri/ /index.html; }
       - server:
          file_name: bar
          vars:
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
      - config:
         file_name: proxy
         vars:
          - proxy_set_header X-Real-IP  $remote_addr
          - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
      - config:
         file_name: upstream
         vars:
          - upstream foo { server 127.0.0.1:8080 weight=10; }
      - config:
         file_name: geo
         vars:
          - geo $local {
              default 0;
              127.0.0.1 1;
            }

Examples
========

1) Install nginx with HTTP directives of choices, but with no sites
configured:

    - hosts: all
      roles:
      - {role: nginx,
         nginx_http_params: ["sendfile on", "access_log /var/log/nginx/access.log"],
                              nginx_sites: none, nginx_configs: none }


2) Install nginx with different HTTP directives than previous example, but no
sites configured.

    - hosts: all
      roles:
      - {role: nginx,
         nginx_http_params: ["tcp_nodelay on", "error_log /var/log/nginx/error.log"],
                              nginx_sites: none, 
                              nginx_configs: none }

Note: Please make sure the HTTP directives passed are valid, as this role
won't check for the validity of the directives. See the nginx documentation
for details.

3) Install nginx and add a site to the configuration.

    - hosts: all

      roles:
      - role: nginx,
        nginx_http_params:
          - sendfile "on"
          - access_log "/var/log/nginx/access.log"
        nginx_sites:
          - server:
             file_name: bar
             vars:
              - listen 8080
              - location / { try_files $uri $uri/ /index.html; }
              - location /images/ { try_files $uri $uri/ /index.html; }
        nginx_configs:
           - config:
               file_name: proxy
               vars:
                - proxy_set_header X-Real-IP  $remote_addr
                - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for

Note: Each site added is represented by list of hashes, and the configurations
generated are populated in /etc/nginx/site-available/, a link is from /etc/nginx/site-enable/ to /etc/nginx/site-available

The file name for the specific site configurtaion is specified in the hash
with the key "file_name", any valid server directives can be added to hash.
Additional configuration are created in /etc/nginx/conf.d/

4) Install Nginx , add 2 sites (different method) and add additional configuration

    ---
    - hosts: all
      roles:
        - role: nginx
          nginx_http_params:
            sendfile: "on"
            access_log: "/var/log/nginx/access.log"
          nginx_sites:
           - server:
              file_name: foo
              vars:
               - listen 8080
               - server_name localhost
               - root /tmp/site1
               - location / { try_files $uri $uri/ /index.html; }
               - location /images/ { try_files $uri $uri/ /index.html; }
           - server:
              file_name: bar
              vars:
               - listen 9090
               - server_name ansible
               - root /tmp/site2
               - location / { try_files $uri $uri/ /index.html; }
               - location /images/ { try_files $uri $uri/ /index.html; }
          nginx_configs:
           - config:
               file_name: proxy
               vars:
                - proxy_set_header X-Real-IP  $remote_addr
                - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for

5) Install Nginx , add 2 sites, add additional configuration and an upstream configuration block

    ---
    - hosts: all
      roles:
        - role: nginx
          nginx_http_params:
            sendfile: "on"
            access_log: "/var/log/nginx/access.log"
          nginx_sites:
           - server:
              file_name: foo
              vars:
               - listen 8080
               - server_name localhost
               - root /tmp/site1
               - location / { try_files $uri $uri/ /index.html; }
               - location /images/ { try_files $uri $uri/ /index.html; }
           - server:
              file_name: bar
              vars:
               - listen 9090
               - server_name ansible
               - root /tmp/site2
               - if ( $host = example.com ) { rewrite ^(.*)$ http://www.example.com$1 permanent; }
               - location / { try_files $uri $uri/ /index.html; }
               - location /images/ { try_files $uri $uri/ /index.html; }
          nginx_configs:
           - config:
               file_name: proxy
               vars:
                - proxy_set_header X-Real-IP  $remote_addr
                - proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for
            - config:
                # Results in:
                # upstream foo_backend {
                #   server 127.0.0.1:8080 weight=10
                # }
                file_name: upstream
                vars:
                 - upstream foo_backend { server 127.0.0.1:8080 weight=10; }

Dependencies
------------

None

License
-------

BSD

Author Information
------------------

Benno Joy


