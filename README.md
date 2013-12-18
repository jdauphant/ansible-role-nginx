nginx
========

This role enables user to install and configure nginx, The user can specify any http paramters they wish to apply thier site.
Also any number of sites can be added with configurations of your choice. 

Requirements
------------

This role requires ansible 1.4 or higher and platform requirements are listed in the metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about them are as follows.

```
nginx_max_clients: 512                                # The max clients allowed
nginx_http_params:                                    # A hash of the http paramters. note that any valid http paramters can be
  sendfile: "on"                                      # added here.
  tcp_nopush: "on"
  tcp_nodelay: "on"
  keepalive_timeout: "65"
  access_log: "/var/log/nginx/access.log"
  error_log: "/var/log/nginx/error.log"

nginx_sites:                                         # A list of hash's that define the servers for nginx, as with http paramters
 - server:                                           # any valid server paramters can be added here.
    file_name: foo
    listen: 8080
    server_name: localhost
    root: "/tmp/site1"
    location1: {name: /, try_files: "$uri $uri/ /index.html"}
    location2: {name: /images/, try_files: "$uri $uri/ /index.html"}
 - server:
    file_name: bar
    listen: 9090
    server_name: ansible
    root: "/tmp/site2"
    location1: {name: /, try_files: "$uri $uri/ /index.html"}
    location2: {name: /images/, try_files: "$uri $uri/ /index.html"}
```

- Examples of using this role 

1) Eg:  Install nginx with http directives of choices, but no sites configured.

```
- hosts: all
  roles:
    - {role: nginx, nginx_http_params: {sendfile: "on", access_log: "/var/log/nginx/access.log"}, nginx_sites: none}
```


1) Eg:  Install nginx with diffrent http directives than previous example, but no sites configured.

```
- hosts: all
  roles:
    - {role: nginx, nginx_http_params: {tcp_nodelay: "on", error_log: "/var/log/nginx/error.log"}, nginx_sites: none}

Note: Please make sure the http directive's passed are valid as this role wont check for the validity of the directives.
```

3) eg: Install nginx and add a site to the configuration.
```
- hosts: all
  roles:
    - {role: nginx, nginx_http_params: {sendfile: "on", access_log: "/var/log/nginx/access.log"}, nginx_sites: [server: {file_name: bar, listen: 8080, location1: {name: /, try_files: "$uri $uri/ /index.html"}, location2: {name: /images/, try_files: "$uri $uri/ /index.html"}}]}
```
Note: Each site added is represented by list of hashes and the configurations generated are populated in /etc/nginx/conf.d/
The file name for the specific site configurtaion is specified in the hash with the key "file_name", any valid server directives
can be added to hash. For location directive add the key "location" suffixed by a unique number, the value for the location is hash, please make sure they are valid location directives.


4) Eg: Install Nginx and add 2 sites (diffrent method)

```

---
- hosts: all
  roles:
    - role: nginx
      nginx_http_params: {sendfile: "on", access_log: "/var/log/nginx/access.log"}
      nginx_sites:
       - server:
          file_name: foo
          listen: 8080
          server_name: localhost
          root: "/tmp/site1"
          location1: {name: /, try_files: "$uri $uri/ /index.html"}
          location2: {name: /images/, try_files: "$uri $uri/ /index.html"}
       - server:
          file_name: bar
          listen: 9090
          server_name: ansible
          root: "/tmp/site2"
          location1: {name: /, try_files: "$uri $uri/ /index.html"}
          location2: {name: /images/, try_files: "$uri $uri/ /index.html"}

 
```

Dependencies
------------

None

License
-------

BSD

Author Information
------------------

Benno Joy


