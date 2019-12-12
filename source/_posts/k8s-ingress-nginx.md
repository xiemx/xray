---
layout: post
title: k8s ingress-nginx动态balance实现解析
published: true
author: xiemx
comments: true
date: 2019-09-16 07:09:52
tags: [ k8s, ingress]
categories:
    - k8s
---
只节选了比较关键的代码，删除了比较多的干扰项。纯属个人理解！！！

#### 1. 初始化balancer.init_worker()，使用balancer.balance()动态获取
```lua
http {
        lua_package_path "/etc/nginx/lua/?.lua;/etc/nginx/lua/vendor/?.lua;/usr/local/lib/lua/?.lua;;";
        init_by_lua_block {
                ok, res = pcall(require, "configuration")
        }

        init_worker_by_lua_block {
                balancer.init_worker()  #####创建定时任务 ngx.timer.every(BACKENDS_SYNC_INTERVAL, sync_backends)
        }
     
###upstream configure
upstream upstream_balancer {
		server 0.0.0.1; # placeholder

		balancer_by_lua_block {
			balancer.balance()
		}
		keepalive 32;
		keepalive_timeout  60s;
		keepalive_requests 100;
	}
```
#### 2. 获取backend信息，balancer.init_worker()
```lua
####https://sourcegraph.com/github.com/kubernetes/ingress-nginx@dd0fe4b458cc5520f25eb8bba25bbe6f0c72ee98/-/blob/rootfs/etc/nginx/lua/balancer.lua?utm_source=share#L223

local configuration = require("configuration")

function _M.init_worker()
  sync_backends() -- when worker starts, sync backends without delay
  local _, err = ngx.timer.every(BACKENDS_SYNC_INTERVAL, sync_backends)
  if err then
    ngx.log(ngx.ERR, string.format("error when setting up timer.every for sync_backends: %s", tostring(err)))
  end
end


local function sync_backends()
  local backends_data = configuration.get_backends_data()

  local new_backends, err = cjson.decode(backends_data)


  local balancers_to_keep = {}
  for _, new_backend in ipairs(new_backends) do
    sync_backend(new_backend)
    balancers_to_keep[new_backend.name] = balancers[new_backend.name]
  end
end
```
```lua
####https://sourcegraph.com/github.com/kubernetes/ingress-nginx@dd0fe4b458cc5520f25eb8bba25bbe6f0c72ee98/-/blob/rootfs/etc/nginx/lua/configuration.lua?utm_source=share#L10:10

local configuration_data = ngx.shared.configuration_data
local certificate_data = ngx.shared.certificate_data

local _M = {}

function _M.get_backends_data()
  return configuration_data:get("backends")
end

function _M.call()
  if ngx.var.request_method ~= "POST" and ngx.var.request_method ~= "GET" then
    ngx.status = ngx.HTTP_BAD_REQUEST
    ngx.print("Only POST and GET requests are allowed!")
    return
  end

  if ngx.var.request_uri == "/configuration/servers" then
    handle_servers()
    return
  end

  if ngx.var.request_uri == "/configuration/general" then
    handle_general()
    return
  end

  if ngx.var.uri == "/configuration/certs" then
    handle_certs()
    return
  end

  if ngx.var.request_uri ~= "/configuration/backends" then ####只接受以上4类型URL
    ngx.status = ngx.HTTP_NOT_FOUND
    ngx.print("Not found!")
    return
  end

  local backends = fetch_request_body()

  local success, err = configuration_data:set("backends", backends)

end
```
```lua
### fetch_request_body()，从此函数可以看出此函数是一个外部调用，可以得出原始的数据来源为外部触发的POST，可以查询Call()函数的引用位置
local function fetch_request_body()
  ngx.req.read_body() ###防止ngx.req.get_body_data()返回nil,显示执行一下
  local body = ngx.req.get_body_data()

  if not body then
    local file_name = ngx.req.get_body_file() ###读取cache file

    local file = io.open(file_name, "rb")
    if not file then
      return nil
    end

    body = file:read("*all")
    file:close()
  end

  return body
end
```
```
####nginx.conf 查看nginx配置文件中显示调用call()函数的位置为当前server切url /configuration 符合函数要求，在查找外部调用的代码（基本可以定位为控制器的逻辑控制）
        server {
                listen unix:/tmp/nginx-status-server.sock;
                set $proxy_upstream_name "internal";

                keepalive_timeout 0;
                gzip off;

                access_log off;

                location /configuration {
                        # this should be equals to configuration_data dict
                        client_max_body_size                    10m;
                        client_body_buffer_size                 10m;
                        proxy_buffering                         off;

                        content_by_lua_block {
                                configuration.call()
                        }
                }
```
#### 3. 通过上面的信息检索，在Ingress中监听pod变化信息，动态调用/configuration/backends, 函数为configureBackends()
```go
####https://github.com/kubernetes/ingress-nginx/blob/ce3e3d51c397ff6a0cd6731cc64360ecdb69ea54/internal/ingress/controller/nginx.go#L982
func configureBackends(rawBackends []*ingress.Backend) error {
	backends := make([]*ingress.Backend, len(rawBackends))

	for i, backend := range rawBackends {
		var service *apiv1.Service
		if backend.Service != nil {
			service = &apiv1.Service{Spec: backend.Service.Spec}
		}
		luaBackend := &ingress.Backend{
			Name:                 backend.Name,
			Port:                 backend.Port,
			SSLPassthrough:       backend.SSLPassthrough,
			SessionAffinity:      backend.SessionAffinity,
			UpstreamHashBy:       backend.UpstreamHashBy,
			LoadBalancing:        backend.LoadBalancing,
			Service:              service,
			NoServer:             backend.NoServer,
			TrafficShapingPolicy: backend.TrafficShapingPolicy,
			AlternativeBackends:  backend.AlternativeBackends,
		}

		var endpoints []ingress.Endpoint
		for _, endpoint := range backend.Endpoints {
			endpoints = append(endpoints, ingress.Endpoint{
				Address: endpoint.Address,
				Port:    endpoint.Port,
			})
		}

		luaBackend.Endpoints = endpoints
		backends[i] = luaBackend
	}

	statusCode, _, err := nginx.NewPostStatusRequest("/configuration/backends", "application/json", backends) ####backends 为request.body,却内容为IP/PORT，以下给出了backend的struct
	if err != nil {
		return err
	}

	if statusCode != http.StatusCreated {
		return fmt.Errorf("unexpected error code: %d", statusCode)
	}

	return nil
}

####Backend struct
type Backend struct {
	Name    string             `json:"name"`
	Service *apiv1.Service     `json:"service,omitempty"`
	Port    intstr.IntOrString `json:"port"`
	SecureCACert resolver.AuthSSLCert `json:"secureCACert"`
	SSLPassthrough bool `json:"sslPassthrough"`
	Endpoints []Endpoint `json:"endpoints,omitempty"`
	SessionAffinity SessionAffinityConfig `json:"sessionAffinityConfig"`
	UpstreamHashBy UpstreamHashByConfig `json:"upstreamHashByConfig,omitempty"`
	LoadBalancing string `json:"load-balance,omitempty"`
	NoServer bool `json:"noServer"`
	TrafficShapingPolicy TrafficShapingPolicy `json:"trafficShapingPolicy,omitempty"`
	AlternativeBackends []string `json:"alternativeBackends,omitempty"`
}
```
#### 4. ngx\_balancer.set\_current_peer()设置backend信息
```lua
####https://sourcegraph.com/github.com/kubernetes/ingress-nginx@dd0fe4b458cc5520f25eb8bba25bbe6f0c72ee98/-/blob/rootfs/etc/nginx/lua/balancer.lua?utm_source=share#L232:13

function _M.balance()
  local balancer = get_balancer()
  if not balancer then
    return
  end

  local peer = balancer:balance()
  if not peer then
    ngx.log(ngx.WARN, "no peer was returned, balancer: " .. balancer.name)
    return
  end

  ngx_balancer.set_more_tries(1)

  local ok, err = ngx_balancer.set_current_peer(peer) ####设置server信息
  if not ok then
    ngx.log(ngx.ERR, string.format("error while setting current upstream peer %s: %s", peer, err))
  end
end


local function get_balancer()
  if ngx.ctx.balancer then
    return ngx.ctx.balancer
  end

  local backend_name = ngx.var.proxy_upstream_name ###获取当前request上下文中共享的变量proxy_upstream_name

  local balancer = balancers[backend_name] ###获取balancers信息由sync_backend()函数定时轮询
  if not balancer then
    return
  end

  if route_to_alternative_balancer(balancer) then
    local alternative_backend_name = balancer.alternative_backends[1]
    ngx.var.proxy_alternative_upstream_name = alternative_backend_name

    balancer = balancers[alternative_backend_name]
  end

  ngx.ctx.balancer = balancer

  return balancer
end
```
```
###nginx.conf
set $proxy_upstream_name    "dev-dev-auto-deploy-5000";
set $proxy_host             $proxy_upstream_name;

proxy_pass http://upstream_balancer;
```