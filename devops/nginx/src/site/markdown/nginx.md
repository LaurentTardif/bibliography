# NGINX 

## NGINX PROXY


Let's start by the directives 


###  Location , with or without /

 If a location is defined by a prefix string that ends with the slash character, and requests are processed by one of proxy_pass, fastcgi_pass, uwsgi_pass, scgi_pass, memcached_pass, or grpc_pass, then the special processing is performed. In response to a request with URI equal to this string, but without the trailing slash, a permanent redirect with the code 301 will be returned to the requested URI with the slash appended. If this is not desired, an exact match of the URI and location could be defined like this:

    location /user/ {
        proxy_pass http://user.example.com;
    }

    location = /user {
        proxy_pass http://login.example.com;
    }

	
### the IF 

In short, Nginx's "if" block effectively creates a (nested) location block and once the "if" condition matches, only the content handler of the inner location block (i.e., the "if" block) will be executed.

The inner block does not has any content handler, ngx_proxy inherits the content handler (that of ngx_proxy) in the outer scope (see src/http/modules/ngx_http_proxy_module.c:2025).
Also the config specified by proxy_pass also gets inherited by the inner "if" block (see src/http/modules/ngx_http_proxy_module.c:2015)

	

### Proxy_pass

	location /pathPrefix/ {
		proxy_pass http://newServer/newPathPrefix;
	}

the part of a normalized request URI matching the location is replaced by a URI specified in the directive.


ie : 
	http://oldServer/pathPrefix/secondPartOfPath will redirect to :
    http://newServer/newPathPrefix/normalized(secondPartOfPath)
	  

	  
### Special case 

When the URI is changed inside a proxied location using the rewrite directive, and this same configuration will be used to process a request (break):

    location /name/ {
        rewrite    /name/([^/]+) /users?name=$1 break;
        proxy_pass http://127.0.0.1;
    }

In this case, the URI specified in the directive is ignored and the full changed request URI is passed to the server. 





### artifactory

	// add ssl entries when https has been set in config
	ssl_certificate      /etc/nginx/ssl/docker.jfrog.com.crt;
	ssl_certificate_key  /etc/nginx/ssl/docker.jfrog.com.key;
	ssl_session_cache shared:SSL:1m;
	ssl_prefer_server_ciphers   on;
	// server configuration
	server {
		listen 443 ssl;
		listen 80 ;
     
		server_name artifactory.jfrog.com;
		if ($http_x_forwarded_proto = '') {
			set $http_x_forwarded_proto  $scheme;
		}
		// Application specific logs
		// access_log /var/log/nginx/artifactory.jfrog.com-access.log timing;
		// error_log /var/log/nginx/artifactory.jfrog.com-error.log;
		rewrite ^/$ /artifactory/webapp/ redirect;
		rewrite ^/artifactory/?(/webapp)?$ /artifactory/webapp/ redirect;

		//################################################################
		### ADDED FOR DOCKER 
		rewrite ^/(v1|v2)/(.*) /artifactory/api/docker/$repo/$1/$2;
		################################################################
	
		chunked_transfer_encoding on;
		client_max_body_size 0;
		location / {
			proxy_read_timeout  900;
		proxy_pass_header   Server;
		proxy_cookie_path   ~*^/.* /;
		if ( $request_uri ~ ^/artifactory/(.*)$ ) {
			proxy_pass          http://<rproxy_artifactory>:8081/artifactory/$1;
		}
		proxy_pass         http://rproxy_artifactory:8081/artifactory/;
		proxy_set_header   X-Artifactory-Override-Base-Url $http_x_forwarded_proto://$host:$server_port/<public context>;
		proxy_set_header    X-Forwarded-Port  $server_port;
		proxy_set_header    X-Forwarded-Proto $http_x_forwarded_proto;
		proxy_set_header    Host              $http_host;
		proxy_set_header    X-Forwarded-For   $proxy_add_x_forwarded_for;
		}
	}
	