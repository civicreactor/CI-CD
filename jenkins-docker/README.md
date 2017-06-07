# How to build your Jenkins container
```bash
cd jenkins-docker/
docker build .
```
# Run your Jenkins container
```bash
docker run -it -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker -v jenkins_home/:/home/jenkins_home <docker-image-identifier>
```

The `-v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/bin/docker` exposes the Docker API and `docker` command executable so they can be used from inside of the Jenkins container to manipulate the Docker daemon on the host.


# Updating the Jenkins plugins
```bash
JENKINS_HOST=username:password@myhost.com:port
curl -sSL "http://$JENKINS_HOST/pluginManager/api/xml?depth=1&xpath=/*/*/shortName|/*/*/version&wrapper=plugins" | perl -pe 's/.*?<shortName>([\w-]+).*?<version>([^<]+)()(<\/\w+>)+/\1 \2\n/g'|sed 's/ /:/' > plugins.txt
```

# Jenkins reverse proxy configuration

```
server {
  listen          80;       # Listen on port 80 for IPv4 requests

  server_name     jenkins.civicreactor.org;

  ignore_invalid_headers off; #pass through headers from Jenkins which are considered invalid by Nginx server.

  location ~ "^/static/[0-9a-fA-F]{8}\/(.*)$" {
    #rewrite all static files into requests to the root
    #E.g /static/12345678/css/something.css will become /css/something.css
    rewrite "^/static/[0-9a-fA-F]{8}\/(.*)" /$1 last;
  }

  location @jenkins {
      sendfile off;
      proxy_pass         http://172.17.0.2:8080;
      proxy_redirect     default;
      proxy_http_version 1.1;

      proxy_set_header   Host             $host;
      proxy_set_header   X-Real-IP        $remote_addr;
      proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
      proxy_max_temp_file_size 0;

      #this is the maximum upload size
      client_max_body_size       10m;
      client_body_buffer_size    128k;

      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;

      proxy_buffer_size          4k;
      proxy_buffers              4 32k;
      proxy_busy_buffers_size    64k;
      proxy_temp_file_write_size 64k;
  }

  location / {
    # Optional configuration to detect and redirect iPhones
    if ($http_user_agent ~* '(iPhone|iPod)') {
      rewrite ^/$ /view/iphone/ redirect;
    }

    try_files $uri @jenkins;
  }
}
```

