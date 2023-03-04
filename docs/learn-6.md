#### 资源对象
Upstream: 上游，指向具体主机服务，可配置负载均衡、健康检查、重试

Consumer: 消费者，指代具体某个人/物，与认证插件捆绑进行认证。

#### 可用架构
route-->service(绑定plugin)-->指向upstream


##### 功能
支持蓝绿发布(撤下组A更新代码，只有组B提供服务)、限速限流、ip白名单

##### 为什么要用到APISIX（api网关）？
微服务体系下，统一API接入和输出。(即一个域名就可以接入所有服务的api)
### 插件开发须知
请求经过网关的不同阶段： init, rewrite, access, balancer, header filter, body filter and log。


#### key-auth插件分析
```
定义插件名称、优先级、schema

local _M = {                    
    version = 0.1,     
    priority = 2500,   
    type = 'auth',  
    name = plugin_name,
    schema = schema,   
    consumer_schema = consumer_schema,
}
```
#### 定义消费者配置格式
```
local consumer_schema = {
    type = "object",
    properties = {
        key = {type = "string"},
    },
    required = {"key"},
}
```
示例：{"key": "auth-one"}

#### 设置从路由获取参数的方式：header 或 query
```
local schema = {
    type = "object",
    properties = {
        header = {
            type = "string",
            default = "apikey",
        },
        query = {
            type = "string",
            default = "apikey",
        },
    },
}
```
示例：
从header获取：curl localhost:9080/web1 -H"apikey: auth-one"
从query获取：http://192.168.1.167:9080/web1?apikey=auth-one

若同时设置，优先取header。

#### 校验用户传入参数合法性
```
function _M.check_schema(conf, schema_type)                                   
    if schema_type == core.schema.TYPE_CONSUMER then                          
        return core.schema.check(consumer_schema, conf)                       
    else                                                                      
        return core.schema.check(schema, conf)                                
    end                                                                       
end
```
#### 若为认证auth阶段，在rewrite阶段执行
```
function _M.rewrite(conf, ctx)                                                
    local key = core.request.header(ctx, conf.header)                         
                                                                              
    if not key then                                                           
        local uri_args = core.request.get_uri_args(ctx) or {}                 
        key = uri_args[conf.query]                                            
    end                                                                       
                                                                              
    if not key then                                                           
        return 401, {message = "Missing API key found in request"}            
    end                                                                       
                                                                              
    local consumer_conf = consumer_mod.plugin(plugin_name)                    
    if not consumer_conf then                                                 
        return 401, {message = "Missing related consumer"}                    
    end                                                                       
                                                                              
    local consumers = lrucache("consumers_key", consumer_conf.conf_version,   
        create_consume_cache, consumer_conf)                                  
                                                                              
    local consumer = consumers[key]                                           
    if not consumer then                                                      
        return 401, {message = "Invalid API key in request"}                  
    end                                                                       
    core.log.info("consumer: ", core.json.delay_encode(consumer))             
                                                                              
    consumer_mod.attach_consumer(ctx, consumer, consumer_conf)                
    core.log.warn("warn : -> hit key-auth rewrite")                           
end
```
#### 热加载插件
```curl 'http://127.0.0.1:9180/apisix/admin/plugins/reload' -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT```

#### 访问认证插件
```curl localhost:9080/web1 -H"apikey: auth-one"```

#### 插件日志优先级
error > warn > info 

#日志将输出到error.log文件
core.log.warn("warn : -> hit key-auth rewrite")

#### conf参数
代表插件配置信息

core.log.warn(core.json.encode(conf))
    打印 =>

{"query":"apikey","disable":false,"header":"apikey"}
