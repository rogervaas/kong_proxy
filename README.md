# NGINX Forward PROXY
If you want to use nginx as the foward proxy server, look no further!!!

### Source Install

First, you will need to already have Cassandra installed. Make sure you have downloaded [Cassandra](http://cassandra.apache.org/download/) and that it is running. 
Then, install kong proxy:

```shell
$ git clone https://github.com/haifengkao/kong_proxy
$ cd kong/

# Install latest Kong and created the development configuration file
$ sudo make dev
```

### Running for development

```shell
# run Kong Proxy with the just created development configuration
$ kong start -c kong_DEVELOPMENT.yml
```

Since you use a configuration file dedicated to development, feel free to customize it as you wish. For example, the one generated by `make dev` includes the following changes: the [`lua_package_path`](https://github.com/openresty/lua-nginx-module#lua_package_path) directive specifies that the Lua modules in your current directory will be used in favor of the system installation. The [`lua_code_cache`](https://github.com/openresty/lua-nginx-module#lua_code_cache) directive being turned off, you can start Kong, edit your local files, and test your code without restarting Kong.

To stop Kong, you will need to specify the configuration file too:

```shell
$ kong stop -c kong_DEVELOPMENT.yml
# or
$ kong reload -c kong_DEVELOPMENT.yml
```

### Configure the proxy plugin

##### Add an api
The kong proxy is a plugin of kong framework.
You need to enable the plugin by:

```shell
$ curl -i -X POST \
  --url http://localhost:8001/apis/ \
  --data 'name=myProxyPlugin' \
  --data 'upstream_url=http://someHostYouLike.com' \
  --data 'request_host=someHostYouLike.com'
```

##### Add kong proxy plugin to the api
```shell
$ curl -i -X POST \
  --url http://localhost:8001/apis/myProxyPlugin/plugins/ \
  --data 'name=kong-proxy' \ 
  --data "config.HostTag=The-Real-Host"
```

##### Send a http request which will be proxied to google.com
```shell
$ curl -i -X GET \
  --url http://localhost:8000/search?q=kong+proxy \
  --header 'Host: someHostYouLike.com' \
  --header 'The-Real-Host: http://google.com' 
```

### Makefile

When developing, you can use the `Makefile` for doing the following operations:

| Name               | Description                                             |
| ------------------:| --------------------------------------------------------|
| `install`          | Install the Kong luarock globally                       |
| `dev`              | Setup your development environment                      |
| `clean`            | Clean your development environment                      |
| `doc`              | Generate the ldoc documentation                         |
| `lint`             | Lint Lua files in `kong/` and `spec/`                   |
| `test`             | Run the unit tests suite                                |
| `test-integration` | Run the integration tests suite                         |
| `test-plugins`     | Run the plugins test suite                              |
| `test-all`         | Run all unit + integration tests at once                |
| `coverage`         | Run all tests + coverage report                         |

### How it works
[Kong](http://getkong.org) provided an extensible framework to write lua plugins for nginx.
The forward proxy functionality is provided by the kong-proxy plugin (kong/plugins/kong-proxy).
The kong-proxy plugin reads the host specified in the 'The-Real-Host' tag and update the upstream url(`ngx.var.backend_url`).
