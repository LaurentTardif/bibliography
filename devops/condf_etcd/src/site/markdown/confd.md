<!-- vscode-markdown-toc -->
* 1. [INSTALLATION](#INSTALLATION)
* 2. [ General Architecture](#GeneralArchitecture)
	* 2.1. [Template file (TOML) for Resources](#TemplatefileTOMLforResources)
	* 2.2. [ The ressource file in template](#Theressourcefileintemplate)
		* 2.2.1. [Template Functions](#TemplateFunctions)
	* 2.3. [Create the template resource](#Createthetemplateresource)
	* 2.4. [Create the template](#Createthetemplate)
	* 2.5. [Advanced Map Traversals](#AdvancedMapTraversals)
	* 2.6. [Example Usage](#ExampleUsage)
	* 2.7. [Complex example](#Complexexample)

<!-- vscode-markdown-toc-config
	numbering=true
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc --># CONFD

<!-- MACRO{toc|section=0|fromDepth=0|toDepth=3} -->


##  1. <a name='INSTALLATION'></a>INSTALLATION

 * Source : https://github.com/kelseyhightower/confd



##  2. <a name='GeneralArchitecture'></a> General Architecture

 * confd is a binary that will feed a resource file, from a template, getting information from a key/value store or the environment.


###  2.1. <a name='TemplatefileTOMLforResources'></a>Template file (TOML) for Resources

Template resources are written in TOML and define a single template resource. Template resources are stored under the */etc/confd/conf.d* directory by default.

**Required**

    dest (string) - The target file.
    keys (array of strings) - An array of keys.
    src (string) - The relative path of a configuration template.

**Optional**

    gid (int) - The gid that should own the file. Defaults to the effective gid.
    mode (string) - The permission mode of the file.
    uid (int) - The uid that should own the file. Defaults to the effective uid.
    reload_cmd (string) - The command to reload config.
    check_cmd (string) - The command to check config. Use {{.src}} to reference the rendered source template.
    prefix (string) - The string to prefix to keys.

**Notes**

When using the reload_cmd feature it's important that the command exits on its own. The reload command is not managed by confd, and will block the configuration run until it exits.
Example

	[template]
	src = "nginx.conf.tmpl"
	dest = "/etc/nginx/nginx.conf"
	uid = 0
	gid = 0
	mode = "0644"
	keys = [
		"/nginx",
	]
	check_cmd = "/usr/sbin/nginx -t -c {{.src}}"
	reload_cmd = "/usr/sbin/service nginx restart"


###  2.2. <a name='Theressourcefileintemplate'></a> The ressource file in template

Templates define a single application configuration template. Templates are stored under the */etc/confd/templates* directory by default.

Templates are written in Go's text/template.

####  2.2.1. <a name='TemplateFunctions'></a>Template Functions

**map**

creates a key-value map of string -> interface{}

	{{$endpoint := map "name" "elasticsearch" "private_port" 9200 "public_port" 443}}

	name: {{index $endpoint "name"}}
	private-port: {{index $endpoint "private_port"}}
	public-port: {{index $endpoint "public_port"}}

specifically useful if you use a sub-template and you want to pass multiple values to it.

**base**

Alias for the path.Base function.

	{{with get "/key"}}
		key: {{base .Key}}
		value: {{.Value}}
	{{end}}

**exists**

Checks if the key exists. Return false if key is not found.

	{{if exists "/key"}}
		value: {{getv "/key"}}
	{{end}}

**get**

Returns the KVPair where key matches its argument. Returns an error if key is not found.

	{{with get "/key"}}
		key: {{.Key}}
		value: {{.Value}}
	{{end}}

**cget**

Returns the KVPair where key matches its argument and the value has been encrypted. Returns an error if key is not found.

	{{with cget "/key"}}
		key: {{.Key}}
		value: {{.Value}}
	{{end}}

**gets**

Returns all KVPair, []KVPair, where key matches its argument. Returns an error if key is not found.

	{{range gets "/*"}}
		key: {{.Key}}
		value: {{.Value}}
	{{end}}

**cgets**

Returns all KVPair, []KVPair, where key matches its argument and the values have been encrypted. Returns an error if key is not found.

	{{range cgets "/*"}}
		key: {{.Key}}
		value: {{.Value}}
	{{end}}

**getv**

Returns the value as a string where key matches its argument or an optional default value. Returns an error if key is not found and no default value given.

	value: {{getv "/key"}}

With a default value

	value: {{getv "/key" "default_value"}}

**cgetv**

Returns the encrypted value as a string where key matches its argument. Returns an error if key is not found.

	value: {{cgetv "/key"}}

**getvs**

Returns all values, []string, where key matches its argument. Returns an error if key is not found.

	{{range getvs "/*"}}
		value: {{.}}
	{{end}}

**cgetvs**

Returns all encrypted values, []string, where key matches its argument. Returns an error if key is not found.

	{{range cgetvs "/*"}}
		value: {{.}}
	{{end}}

**getenv**

Wrapper for os.Getenv. Retrieves the value of the environment variable named by the key. It returns the value, which will be empty if the variable is not present. Optionally, you can give a default value that will be returned if the key is not present.

	export HOSTNAME=`hostname`

	hostname: {{getenv "HOSTNAME"}}

With a default value

	ipaddr: {{getenv "HOST_IP" "127.0.0.1"}}

**datetime**

Alias for time.Now

	# Generated by confd {{datetime}}

Outputs:

	# Generated by confd 2015-01-23 13:34:56.093250283 -0800 PST


	# Generated by confd {{datetime.Format "Jan 2, 2006 at 3:04pm (MST)"}}

Outputs:

	# Generated by confd Jan 23, 2015 at 1:34pm (EST)

	See the time package for more usage: http://golang.org/pkg/time/

**split**

Wrapper for strings.Split. Splits the input string on the separating string and returns a slice of substrings.

	{{ $url := split (getv "/deis/service") ":" }}
		host: {{index $url 0}}
		port: {{index $url 1}}

**toUpper**

Alias for strings.ToUpper Returns uppercased string.

	key: {{toUpper "value"}}

**toLower**

Alias for strings.ToLower. Returns lowercased string.

	key: {{toLower "Value"}}

**json**

Returns an map[string]interface{} of the json value.

**lookupSRV**

Wrapper for net.LookupSRV. The wrapper also sorts the SRV records alphabetically by combining all the fields of the net.SRV struct to reduce unnecessary config reloads.

	{{range lookupSRV "mail" "tcp" "example.com"}}
		target: {{.Target}}
		port: {{.Port}}
		priority: {{.Priority}}
		weight: {{.Weight}}
	{{end}}

**base64Encode**

Returns a base64 encoded string of the value.

	key: {{base64Encode "Value"}}

**base64Decode**

Returns the string representing the decoded base64 value.

	key: {{base64Decode "VmFsdWU="}}

**Add keys to etcd**

	etcdctl set /services/zookeeper/host1 '{"Id":"host1", "IP":"192.168.10.11"}'
	etcdctl set /services/zookeeper/host2 '{"Id":"host2", "IP":"192.168.10.12"}'


###  2.3. <a name='Createthetemplateresource'></a>Create the template resource

	[template]
	src = "services.conf.tmpl"
	dest = "/tmp/services.conf"
	keys = [
		"/services/zookeeper/"
	]

###  2.4. <a name='Createthetemplate'></a>Create the template

	{{range gets "/services/zookeeper/*"}}
	{{$data := json .Value}}
		id: {{$data.Id}}
		ip: {{$data.IP}}
	{{end}}


###  2.5. <a name='AdvancedMapTraversals'></a>Advanced Map Traversals

Once you have parsed the JSON, it is possible to traverse it with normal Go template functions such as index.

A more advanced structure, like this:

	{
		"animals": [
			{"type": "dog", "name": "Fido"},
			{"type": "cat", "name": "Misse"}
		]
	}

It can be traversed like this:

	{{$data := json (getv "/test/data/")}}
		type: {{ (index $data.animals 1).type }}
		name: {{ (index $data.animals 1).name }}
	{{range $data.animals}}
	{{.name}}
	{{end}}

**jsonArray**

Returns a []interface{} from a json array such as ["a", "b", "c"].

	{{range jsonArray (getv "/services/data/")}}
		val: {{.}}
	{{end}}

**ls**

Returns all subkeys, []string, where path matches its argument. Returns an empty list if path is not found.

	{{range ls "/deis/services"}}
		value: {{.}}
	{{end}}

**lsdir**

Returns all subkeys, []string, where path matches its argument. It only returns subkeys that also have subkeys. Returns an empty list if path is not found.

	{{range lsdir "/deis/services"}}
		value: {{.}}
	{{end}}

**dir**

Returns the parent directory of a given key.

	{{with dir "/services/data/url"}}
		dir: {{.}}
	{{end}}

**join**

Alias for the strings.Join function.

	{{$services := getvs "/services/elasticsearch/*"}}
	services: {{join $services ","}}

**replace**

Alias for the strings.Replace function.

	{{$backend := getv "/services/backend/nginx"}}
	backend = {{replace $backend "-" "_" -1}}

**lookupIP**

Wrapper for net.LookupIP function. The wrapper also sorts (alphabeticaly) the IP addresses. This is crucial since in dynamic environments DNS servers typically shuffle the addresses linked to domain name. And that would cause unnecessary config reloads.

	{{range lookupIP "some.host.local"}}
		server {{.}};
	{{end}}

**atoi**

Alias for the strconv.Atoi function.

	{{seq 1 (atoi (getv "/count"))}}


###  2.6. <a name='ExampleUsage'></a>Example Usage

	etcdctl set /nginx/domain 'example.com'
	etcdctl set /nginx/root '/var/www/example_dotcom'
	etcdctl set /nginx/worker_processes '2'
	etcdctl set /app/upstream/app1 "10.0.1.100:80"
	etcdctl set /app/upstream/app2 "10.0.1.101:80"

	/etc/confd/templates/nginx.conf.tmpl

	worker_processes {{getv "/nginx/worker_processes"}};

	upstream app {
	{{range getvs "/app/upstream/*"}}
		server {{.}};
	{{end}}
	}

	server {
		listen 80;
		server_name www.{{getv "/nginx/domain"}};
		access_log /var/log/nginx/{{getv "/nginx/domain"}}.access.log;
		error_log /var/log/nginx/{{getv "/nginx/domain"}}.log;

		location / {
			root              {{getv "/nginx/root"}};
			index             index.html index.htm;
			proxy_pass        http://app;
			proxy_redirect    off;
			proxy_set_header  Host             $host;
			proxy_set_header  X-Real-IP        $remote_addr;
			proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
		}
	}

	Output: /etc/nginx/nginx.conf

	worker_processes 2;

	upstream app {
		server 10.0.1.100:80;
		server 10.0.1.101:80;
	}

	server {
		listen 80;
		server_name www.example.com;
		access_log /var/log/nginx/example.com.access.log;
		error_log /var/log/nginx/example.com.error.log;

		location / {
			root              /var/www/example_dotcom;
			index             index.html index.htm;
			proxy_pass        http://app;
			proxy_redirect    off;
			proxy_set_header  Host             $host;
			proxy_set_header  X-Real-IP        $remote_addr;
			proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
		}
	}

###  2.7. <a name='Complexexample'></a>Complex example

This examples show how to use a combination of the templates functions to do nested iteration.
	Add keys to etcd

	etcdctl mkdir /services/web/cust1/
	etcdctl mkdir /services/web/cust2/
	etcdctl set /services/web/cust1/2 '{"IP": "10.0.0.2"}'
	etcdctl set /services/web/cust2/2 '{"IP": "10.0.0.4"}'
	etcdctl set /services/web/cust2/1 '{"IP": "10.0.0.3"}'
	etcdctl set /services/web/cust1/1 '{"IP": "10.0.0.1"}'

Create the template resource

	[template]
	src = "services.conf.tmpl"
	dest = "/tmp/services.conf"
	keys = [
		"/services/web"
	]

Create the template

	{{range $dir := lsdir "/services/web"}}
	upstream {{base $dir}} {
		{{$custdir := printf "/services/web/%s/*" $dir}}{{range gets $custdir}}
			server {{$data := json .Value}}{{$data.IP}}:80;
		{{end}}
	}

	server {
		server_name {{base $dir}}.example.com;
		location / {
			proxy_pass {{base $dir}};
		}
	}
	{{end}}

Output:/tmp/services.conf

	upstream cust1 {
		server 10.0.0.1:80;
		server 10.0.0.2:80;
	}

	server {
		server_name cust1.example.com;
		location / {
			proxy_pass cust1;
		}
	}

	upstream cust2 {
		server 10.0.0.3:80;
		server 10.0.0.4:80;
	}

	server {
		server_name cust2.example.com;
		location / {
			proxy_pass cust2;
		}
	}
