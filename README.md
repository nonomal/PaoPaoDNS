# PaoPao DNS docker
![PaoPaoDNS](img.jpg)    
![pull](https://img.shields.io/docker/pulls/sliamb/paopaodns.svg) ![size](https://img.shields.io/docker/image-size/sliamb/paopaodns)   
![Docker Platforms](https://img.shields.io/badge/platforms-linux%2F386%20%7C%20linux%2Famd64%20%7C%20linux%2Farm%2Fv6%20%7C%20linux%2Farm%2Fv7%20%7C%20linux%2Farm64%2Fv8%20%7C%20linux%2Fppc64le%20%7C%20%20linux%2Fs390x-blue)  
泡泡DNS是一个能一键部署递归DNS的docker镜像，它使用了unbound作为递归服务器程序，使用Redis作为底层缓存，此外针对China大陆，还有智能根据CN分流加密查询的功能，也可以自定义分流列表，可以自动更新IP库，分流使用了mosdns程序，加密查询使用dnscrypt程序，针对IPv4/IPv6双栈用户也有优化处理。   
泡泡DNS适合的使用场景：  
- 场景一：仅作为一个纯粹准确的递归DNS服务器，作为你其他DNS服务程序的上游，替代`114.114.114.114`,`8.8.8.8.8`等公共DNS上游
- 场景二：作为一个局域网内具备CN智能分流、解决污染问题和IPv6双栈优化的DNS服务器，或者你的局域网已经从IP层面解决了“科学”的问题，需要一个能智能分流的DNS服务器。  
##### 如果对你有帮助，欢迎点`Star`，如果需要关注更新，可以点`Watch`。

## [→详细说明《为啥需要递归DNS》/运行逻辑](https://blog.03k.org/post/paopaodns.html)
## [更新日志](https://github.com/kkkgo/PaoPaoDNS/discussions/categories/%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97)
## 使用方法
简单来说，那么你可以运行：  
```shell
#拉取最新的docker镜像
docker pull sliamb/paopaodns:latest
#假设你的数据要放在/home/mydata
docker run -d \
--name paopaodns \
-v /home/mydata:/data \
-e CNAUTO=yes \
--restart always \
-p 53:53/tcp -p 53:53/udp \
sliamb/paopaodns
```
如果你需要容器运行在同一个局域网段而不是单独映射端口，除了一些NAS有现成的界面点点点，原生docker你可以考虑使用macvlan如下的配置(假设你的网络是192.168.1.0/24)：  
```shell
# 启用eth0网卡混杂模式
ip link set eth0 promisc on
# 创建macvlan网络
docker network create -d macvlan --subnet=192.168.1.0/24 --gateway=192.168.1.1 -o parent=eth0 macvlan_eth0
#拉取最新的docker镜像
docker pull sliamb/paopaodns:latest
# 运行容器并指定IP
docker run -d \
--name paopaodns \
-v /home/mydata:/data \
-e CNAUTO=yes \
--restart always \
--network macvlan_eth0 --ip 192.168.1.8 \
sliamb/paopaodns
```
***如果你的网络端口没有冲突，也可以考虑使用docker host网络模式以获得最佳性能。***   
*如条件允许建议使用**docker compose**部署*    
如果你的网络环境访问Docker Hub镜像有困难，***可以尝试使用public.ecr.aws镜像:***    
- 示例： `docker pull public.ecr.aws/sliamb/paopaodns`  
- 示例： `docker run -d public.ecr.aws/sliamb/paopaodns`  


验证你的递归DNS正常运行(假设你的容器IP是192.168.1.8)，可以执行以下命令：   
```cmd
>nslookup -type=TXT whoami.ds.akahelp.net 192.168.1.8
服务器:  PaoPaoDNS,blog.03k.org
Address:  192.168.1.8

非权威应答:
whoami.ds.akahelp.net   text =

        "ns"
        "116.31.123.234"  #连接权威DNS服务器的IP=你的宽带IP
Linux可使用dig命令：  
dig whoami.ds.akahelp.net @192.168.1.8 txt -p53
```  
或者，你可以使用03k.org的服务：  
```cmd
>nslookup whoami.03k.org 192.168.1.8
服务器:  PaoPaoDNS,blog.03k.org
Address:  192.168.1.8

非权威应答:
名称:    whoami.03k.org
Address:  116.31.123.234 #连接权威DNS服务器的IP=你的宽带IP
```
如果返回的IP和你宽带的出口IP一致的话，说明你的递归DNS服务正常运作了。 
   
***搭建完请简单验证所有DNS组件是否工作正常：***  
```rust
# 在容器内置执行 test.sh
docker exec paopaodns test.sh
# 如果执行后输出 ALL TEST PASS，则所有组件都工作正常。
# 如果显示 FAIL，可以执行 debug.sh 进一步分析原因。
```   
同时你可以查阅[更新日志](https://github.com/kkkgo/PaoPaoDNS/discussions/categories/%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97)的最新版本公告时间，检查输出的镜像版本时间是否大于等于当前最新版本。   
需要注意的是，如果你的网络有“自动分流IP”的功能，请把容器的IP加入不分流的名单，因为权威DNS需要准确的IP去判断，IP分流会影响权威DNS的判断。此外，一些软路由存在劫持DNS请求的情况，解决办法参见[这个issue](https://github.com/kkkgo/PaoPaoDNS/issues/2#issuecomment-1504708367)。    
***[DNS hijack]DNS劫持算是经常问的高频问题了，[请参考](https://github.com/kkkgo/PaoPaoDNS/discussions/111#discussioncomment-8872824)***   

## 参数说明
环境变量参数如下：  
环境变量|默认值|可用值|
-|-|-|
CNAUTO|`yes`|`yes`,`no`|
DNSPORT|`53`|端口值|
DNS_SERVERNAME|`PaoPaoDNS,blog.03k.org`|不含空格的英文字符串|
SERVER_IP|空，非必须。|IP地址，如`10.10.10.8`|
SOCKS5|空，非必须。|如：`10.10.10.8:7890`|
TZ|`Asia/Shanghai`|tzdata时区值|
UPDATE|`weekly`|`no`,`daily`,`weekly`,`monthly`|
IPV6|`no`|`no`,`yes`,`only6`,`yes_only6`,`raw`|
CNFALL|`yes`|`no`,`yes`|
EXPIRED_FLUSH|`yes`|`no`,`yes`|
CUSTOM_FORWARD|空，可选功能|`IP:PORT`,如`10.10.10.3:53`|
CUSTOM_FORWARD_TTL|`0`|`1-604800`|
AUTO_FORWARD|`no`|`no`,`yes`|
AUTO_FORWARD_CHECK|`yes`|`no`,`yes`|
USE_MARK_DATA|`no`|`no`,`yes`|
RULES_TTL|`0`|`1-604800`|
CN_TRACKER|`yes`|`no`,`yes`|
USE_HOSTS|`no`|`no`,`yes`|
HTTP_FILE|`no`|`no`,`yes`|
SAFEMODE|`no`|`no`,`yes`|
ADDINFO|`no`|`no`,`yes`|
SHUFFLE|`no`|`no`,`yes`,`lite`,`trnc`|
QUERY_TIME|`2000ms`|`time.Duration`|

用途说明：
- CNAUTO：是否开启CN大陆智能分流，如果位于境外可配置为no。当`CNAUTO=no`时，除递归以外的功能（包括规则/列表等）将不会工作。
- DNSPORT：设置DNS服务器端口，仅在CNAUTO=no时生效
- DNS_SERVERNAME：DNS的服务器名称，你使用windows的nslookup的时候会看到它。    
- SERVER_IP：指定DNS服务器的外部IP。假设你的DNS容器是宿主`10.10.10.4`映射出来的端口而不是独立的IP，设置该项为`10.10.10.4`可以让你看到正确的`DNS_SERVERNAME`。同时会设定域名`paopao.dns`指向该IP地址`10.10.10.4`，可配合其他服务使用。         
- SOCKS5：为分流非CN IP的域名优先使用SOCKS5查询（如`10.10.10.8:7890`，强制使用socks5查询则加上@，比如`@10.10.10.8:7890`），但没有也能查，非必须项。仅在CNAUTO=yes时生效。SOCKS5初始化会有大概3分钟的延迟连接测试过程，期间的解析结果并非最优延迟。  
- TZ: 设置系统的运行时区，仅影响输出日志不影响程序运行
- UPDATE: 检查更新根域数据和GEOIP数据的频率,no不检查,其中GEOIP更新仅在CNAUTO=yes时生效。注意：`daily`,`weekly`,`monthly`分别为alpine默认定义的每天凌晨2点、每周6凌晨3点、每月1号凌晨5点。更新数据后会瞬间完成重载。
- IPV6： 仅在CNAUTO=yes时生效，是否返回IPv6的解析结果，默认为no，如果没有IPv6环境，选择no可以节省内存。设置为yes返回IPv6的查询（为分流优化，非大陆双栈域名仅返回A记录）。如果设置为`only6`，则只对IPv6 only的域名返回IPv6结果。如果设置为`yes_only6`，则对大陆域名返回IPv6的解析结果（相当于`yes`），对非大陆域名只对IPv6 only的域名返回IPv6结果（相当于`only6`）。如果设置为`raw`，则不对IPv6结果做任何处理，直接返回原始记录。      
- CNFALL: 仅在CNAUTO=yes时生效，在遇到本地递归网络质量较差的时候，递归查询是否回退到转发查询，默认为yes。配置为no可以保证更实时准确的解析，但要求网络质量稳定（尽量减少nat的层数），推荐部署在具备公网IP的一级路由下的时候设置为no； 配置为yes可以兼顾解析质量和网络质量的平衡，保证长期总体的准确解析的同时兼顾短时间内网络超时的回退处理。    
- EXPIRED_FLUSH: 该选项为`yes`，且在`CNAUTO`、`CNFALL`为`yes`时生效。该选项默认值为`yes`。当开启该选项时，将会主动监测递归结果中出现的乐观缓存，在乐观缓存返回后数秒后检查是否成功递归刷新了新的解析结果，如果递归失败（由两次ttl记录差值对比），将会主动回收清除该缓存。开启该选项可以有效避免乐观缓存因网络连接性不稳定而一直滞留过期记录的问题，提高DNS解析结果的实时性。     
- CUSTOM_FORWARD: 仅在CNAUTO=yes时生效，将`force_forward_list.txt`内的域名列表转发到到`CUSTOM_FORWARD`DNS服务器。该功能可以配合第三方旁网关的fakeip，域名嗅探sniffing等特性完成简单的域名分流效果。    
- CUSTOM_FORWARD_TTL：该项设置的值大于0的时候生效，设定CUSTOM_FORWARD的ttl的最小值。  
- AUTO_FORWARD：仅在CNAUTO=yes时生效，配合`CUSTOM_FORWARD`功能使用，默认值为no，当设置为yes的时候，解析非CN大陆IP的域名将会直接转发到`CUSTOM_FORWARD`。       
- AUTO_FORWARD_CHECK：在`AUTO_FORWARD=yes`时，转发前是否检查域名是否有效，避免产生无效查询。默认值为yes，设置为no则不检查。       
- USE_MARK_DATA：该项默认值为no，当设置为yes的时候，将会自动更新下载预先标记处理的全球百万域名库，在判断大陆分流的时候优先使用该数据，该功能仅标记数据，后续如何处理取决你的设置（比如默认分流或者自动转发）。域名数据库来源于`paopao-pref`项目定期更新。该功能：  
  - 优点：可以优化DNS泄漏问题、提供更快速精准高效的分流
  - 缺点：会占用更多内存，增加容器启动时间
- RULES_TTL：该项设置的值大于0的时候生效，将`/data/force_ttl_rules.txt`里面指定的域名转发到指定的DNS服务器，并修改其TTL值为`RULES_TTL`。该功能仅对A记录和AAAA记录生效，其他记录请参考*进阶自定义示例*一节。该功能可以适用于多种场景，比如想实现在异地的网络访问回家的DDNS域名的结果更实时一点，你可以把`RULES_TTL`设置为一个较低的值，然后把你的DDNS域名指定转发到对应的权威DNS服务器（也就是whois信息的NS服务器对应的IP地址，注意不要CNAME嵌套）即可。`force_ttl_rules`的规则格式为域名@服务器:端口，以下都是合法的格式：
```yaml
# whois info 03k.org:
# Name Servers:
# cold.dnspod.net(129.211.176.224)
# sunfish.dnspod.net(112.80.181.45)

cncheck.03k.org@129.211.176.224
cncheck.03k.org@129.211.176.224:53
cncheck.03k.org@129.211.176.224,112.80.181.45
cncheck.03k.org@129.211.176.224:53,112.80.181.45:53
cncheck.03k.org@129.211.176.224,112.80.181.45:53

# 注意，在该示例中，cncheck.03k.org和其子域名比如www.cncheck.03k.org都会被转发。
```  
此外，`RULES_TTL`功能也可以直接指定某个域名的A记录或者AAAA记录，或者“CNAME”到另一个域名。格式使用域名@@记录或者域名@@@记录，以下都是合法的格式：
```yaml
# 重定向www.qq.com
www.qq.com@@1.2.3.4
www.qq.com@@5.6.7.8 #可以指定多项记录
www.qq.com@@2404:6800:4008:c06::99

# CNAME www.qq.com 到qq.03k.org
www.qq.com@@qq.03k.org

# 注意，使用@@为子域名匹配，上述示例会匹配*.www.qq.com和www.qq.com


# 如果需要精确匹配，可以使用@@@：
www.qq.com@@@1.2.3.4
www.qq.com@@@2404:6800:4008:c06::99
www.qq.com@@@qq.03k.org
```

- CN_TRACKER：仅在CNAUTO=yes时生效，默认值为yes，当设置为yes的时候，强制`trackerslist.txt`里面tracker的域名走dnscrypt解析。更新数据的时候会自动下载最新的trakcerlist。该功能在一些场景比较有用，比如`AUTO_FORWARD`配合fakeip的时候可以避免使用fakeip连接tracker。       
- USE_HOSTS: 当设置为yes的时候，在启动时读取容器/etc/hosts文件。可以配合docker的`-add-hosts`或者docker compose的`extra_hosts`使用。仅在CNAUTO=yes时生效。         
- HTTP_FILE: 当设置为yes的时候，会启动一个7889端口的http静态文件服务器映射`/data`目录。你可以利用此功能与其他服务程序共享文件配置。         
- SAFEMODE： 安全模式，仅作调试使用，内存环境存在问题无法正常启动的时候尝试启用。   
- ADDINFO： 默认为`no`,设置为`yes`时，在DNS查询结果中增加`ADDITIONAL SECTION`的调试信息，如结果来源、查询延迟、失败原因等，使用dig命令就可以实时追踪域名结果来源，详情参考更新日志( https://github.com/kkkgo/PaoPaoDNS/discussions/61 )。该功能仅对`CNAUTO=yes`生效。
- SHUFFLE 默认为`no`,设置为`yes`时，对解析的结果进行洗牌实现`Round-robin DNS`（注：SHUFFLE功能是对每次查询都进行洗牌输出。即使设置为no，在DNS的ttl过期后重新提供的DNS记录本身是经过unbound洗牌过的）。当设置为`lite`，返回精简的仅与请求类型匹配的回应，参考更新日志( https://github.com/kkkgo/PaoPaoDNS/discussions/108 )；当设置为`trnc`，在`lite`选项的基础之上，如果返回的记录大于3个，则每次洗牌完成后仅在ttl有效期内输出3个随机记录，参考更新日志( https://github.com/kkkgo/PaoPaoDNS/discussions/109 )    
- QUERY_TIME：限制DNS转发最大时间，仅作调试使用，随意更改此值会导致你查不到DNS结果。     

可映射TCP/UDP|端口用途
|-|-|
53|提供DNS服务的端口，在CNAUTO=no时数据直接来自unbound，CNAUTO=yes时数据来自mosdns
5301|在CNAUTO=yes时，递归unbound的端口，可用于dig调试
5302|在CNAUTO=yes时，原生dnscrypt服务端口，可用于dig调试
5303|在CNAUTO=yes时并设置了SOCKS5时，走SOCKS5的dnscrypt服务端口，可用于dig调试
5304|在CNAUTO=yes时，dnscrypt的底层unbound实例缓存，可用于dig调试或者fakeip网关的上游  
7889|HTTP_FILE=yes时，http静态文件服务器端口

挂载共享文件夹`/data`目录文件说明：存放redis数据、IP库、各种配置文件，在该目录中修改配置文件会覆盖脚本参数，如果你不清楚配置项的作用，**请不要删除任何注释**。如果修改任何配置出现了异常，把配置文件删除，重启容器即可生成默认文件。   
注：[群晖等挂载权限问题参考](https://github.com/kkkgo/PaoPaoDNS/discussions/52)   
- `redis.conf`：redis服务器配置模板文件，修改它将会覆盖redis运行参数。除了调试用途，一般强烈建议不修改它。容器版本更新将会覆盖该文件。  
- `redis_dns_v2.rdb`：redis的缓存文件，容器重启后靠它读取DNS缓存。刚开始使用的时候因为递归DNS有一个积累的过程，一开始查询会比较慢(设置了CNFALL=no的话，如果CNFALL=yes查询速度不会低于公共DNS)，等到这个文件体积起来了就很流畅了。容器版本更新不会覆盖该文件。    
注意：redis_dns_v2.rdb文件生成需要累积达到redis的最持久化要求，取决于`redis.conf`的配置，默认最低2小时后才会进行一次持久化操作。如果你升级容器的镜像，可以删除其他所有配置文件而保留这个rdb文件。           
- `unbound.conf`：Unbound递归DNS的配置模板文件，除了调试用途，一般不要修改它。容器版本更新将会覆盖该文件。     
- `unbound_custom.conf`：Unbound的自定义配置文件，里面内置了一些高级自定义的示例。容器版本更新不会覆盖该文件。     
**以下文件仅在开启CNAUTO功能时出现：**  
- `dnscrypt-resolvers`文件夹：储存dnscrypt服务器信息和签名，自动动态更新。容器版本更新将会覆盖该文件。  
- `Country-only-cn-private.mmdb`：CN IP数据库，自动更新将会覆盖此文件。容器版本更新将会覆盖该文件。  
- `global_mark.dat`：`USE_MARK_DATA`功能的数据库，自动更新将会覆盖此文件。容器版本更新将会覆盖该文件。  
- `dnscrypt.toml`：dnscrypt配置模板文件，修改它将会覆盖dnscrypt运行参数。除了调试用途，一般不修改它。容器版本更新将会覆盖该文件。   
- `force_forward_list.txt`： 仅在配置`CUSTOM_FORWARD`有效值时生效，强制转发到`CUSTOM_FORWARD`DNS服务器的域名列表，容器版本更新不会覆盖该文件。一行一条，语法规则如下：  
以`domain:`开头域匹配: `domain:03k.org`会匹配自身`03k.org`，以及其子域名`www.03k.org`, `blog.03k.org`等。   
以`full:`开头，完整匹配，`full:03k.org` 只会匹配自身。完整匹配优先级更高。     
以`regexp:`开头，正则匹配，如`regexp:.+\.03k\.org$`。[Go标准正则](https://github.com/google/re2/wiki/Syntax)。   
以`keyword:`开头匹配域名关键字，如以`keyword: 03k.org`会匹配到`www.03k.org.cn`   
尽量避免使用`regexp/keyword`会消耗更多资源。域名表达式省略前缀则为`domain:`。同一文本内匹配优先级：`full > domain > regexp > keyword`     
- `force_dnscrypt_list.txt`：强制使用dnscrypt加密查询结果的域名列表，匹配规则同上。容器版本更新不会覆盖该文件。   
- `force_recurse_list.txt`：强制使用本地递归服务器查询的域名列表，*一般不会用到该list，强制递归的域名不会被生效CNFALL功能*，匹配规则同上。容器版本更新不会覆盖该文件。   
- `force_ttl_rules.txt`: 参见`RULES_TTL`功能。修改将实时重载生效。容器版本更新不会覆盖该文件。   
- 修改`force_forward_list.txt`或`force_dnscrypt_list.txt`或`force_recurse_list.txt`或`force_ttl_rules.txt`将会实时重载生效。
- 文本匹配优先级`(custom_mod功能seq: top)`>`force_forward_list` > `force_dnscrypt_list` > `force_recurse_list` > `force_ttl_rules`>`(custom_mod功能seq: list)`>`其他自动分流逻辑`。
- **注意事项**：由于跨平台系统差异，不建议使用Windows自带记事本编辑。如果list出现了问题无法读取或者无法生效，可以直接删除list文件，重启容器会自动重建默认的list。如果你想解析的域名位于境外，并且没有境内CDN，而你又想获取原始记录（与`force_forward_list.txt`或者使用`AUTO_FORWARD`功能获取到的解析记录区分开），那么你应该把域名加进`force_dnscrypt_list.txt`而不是`force_recurse_list.txt`，因为基于个人网络环境差异，递归服务器位于境外的域名存在递归失败的可能。*`force_recurse_list.txt`的应用场景一般应仅限于特殊域名递归调试，大部分场景都不适用于`force_recurse_list.txt`。* 此外，你可以根据`文本匹配优先级`灵活设置同一个域名子域名走不同的list。（[参考](https://github.com/kkkgo/PaoPaoDNS/discussions/122) ）。         
- `trackerslist.txt`：bt trakcer列表文件，开启`CN_TRACKER`功能会出现，会增量自动更新，[更新数据来源](https://github.com/kkkgo/all-tracker-list) ，你也可以添加自己的trakcer到这个文件(或者向[该项目](https://github.com/kkkgo/all-tracker-list)提交)，更新的时候会自动合并。修改将实时重载生效。容器版本更新不会覆盖该文件。   
- `mosdns.yaml`：mosdns的配置模板文件，修改它将会覆盖mosdns运行参数。除了调试用途，一般强烈建议不修改它。容器版本更新将会覆盖该文件。   
- `custom_env.ini`可以自定义环境变量，会覆盖在容器在启动时的环境变量。在容器启动后修改该文件将会导致MosDNS重载，但在容器启动后修改的环境变量不会影响已经启动的其他组件。配置的格式为`key="value"`（注意英文双引号），错误格式的环境变量将会被忽略加载。容器版本更新不会覆盖该文件。  
- `custom_mod.yaml`可以自定义一些高级功能，参见下面的`custom_mod.yaml`文件说明。错误的配置可能导致服务运行异常。需要重启容器应用配置。容器版本更新不会覆盖该文件。   
**custom_mod.yaml配置说明**  
```yaml
# yaml配置格式请注意空格缩进和冒号，错误的配置将不会被加载。
# Zones可以配置指定域名转发。可以配置多组。
# 与`RULES_TLL`等功能不同，Zones配置的域名转发优先级默认最高，并且可以转发所有记录类型。
Zones:
 - zone: company.local
   dns: udp://10.10.10.3:53,udp://10.10.10.4:53
   ttl: 0
   seq: top
   socks5: no
# - zone: 此处填转发的域名。也可以是子域名，或者后缀。
#   dns: 可以逗号分隔指定多个DNS服务器、udp/tcp协议、端口。
#        指定超过3个DNS服务器将随机选择3个。
#   ttl: 指定该域名的最大ttl值。当设置非0的时候生效。
#        设置为0为不修改原来的ttl。
#   seq: top  #缺省选项，优先级最高，直接进行转发所有类型记录
#        top6 #与top一样但应用全局的IPv6设置
#        list #优先级最低，在匹配所有list后匹配
#   socks5: 可以配置为yes或者no，是否使用socks5代理来查询。
#           仅支持代理tcp协议的dns服务器。
 - zone: .corp
   dns: udp://10.10.10.3:53,udp://10.10.10.4:53
   ttl: 60
   seq: top6
   socks5: no
 - zone: ddns.example.com
   dns: tcp://172.64.32.176:53,tcp://108.162.192.176:53
   ttl: 3
   seq: list
   socks5: yes
# Swaps可以指定某个IP/CIDR段的解析结果替换为指定变量的结果。
# 以最终解析结果为准匹配。与Zones格式类似可以配置多组。
Swaps:
 - env_key: test_ip
   cidr_file: "/data/test_cidr.txt"
# env_key：配置指定变量的解析结果。可以配合custom_env.ini使用。
# cidr_file: 配置指定IP/CIDR段的文本文件。格式为每行一个IP/CIDR段。
# 注意：如果env_key或者cidr_file配置出错，容器日志会报错并忽略替换。
```
注：`Swaps`应用场景参考：[替换指定IP段的解析结果为指定IP](https://github.com/kkkgo/PaoPaoDNS/discussions/57 )   
### 进阶自定义示例

1. 在企业内可能需要的一个功能，就是需要和AD域整合，转发指定域名到AD域服务器的方法：
打开`/data/custom_mod.yaml`编辑：
```yaml
```yaml
#Active Directory Forward Example
# 在这个示例中，你公司的AD域名为company.local，有几个AD域DNS服务器。
Zones:
 - zone: company.local
   dns: 10.111.222.11,10.111.222.12,10.111.222.13
```

2. 添加微软KMS服务器SRV记录
```yaml
#Example of setting up SRV records for KMS server VLMCS.
#假设你的内网后缀是.lan，KMS服务器地址是192.168.1.2或者kms.ad.local

server:
    local-zone: "_vlmcs._tcp.lan." static
    local-data: "_vlmcs._tcp.lan. IN SRV 0 0 1688 kms.ad.local."
    local-data: "_vlmcs._tcp.lan. IN SRV 0 0 1688 192.168.1.2."

```

如果有其他高级的自定义需求，欢迎在[discussions](https://github.com/kkkgo/PaoPaoDNS/discussions)里面参与讨论。   

## 附赠：PaoPao-Pref
这是一个让DNS服务器预读取缓存或者压力测试的简单工具，配合[PaoPaoDNS](https://github.com/kkkgo/PaoPaoDNS)使用可以快速生成`redis_dns_v2.rdb`缓存。从指定的文本读取域名列表并查询A/AAAA记录，docker镜像默认自带了全球前100万热门域名(经过无效域名筛选)。     
详情：https://github.com/kkkgo/PaoPao-Pref    

## 相关项目：PaoPaoGateWay
PaoPao GateWay是一个体积小巧、稳定强大的FakeIP网关，支持`Full Cone NAT` ，支持多种方式下发配置，支持多种出站方式，包括自定义socks5、自定义yaml节点、订阅模式和自由出站，支持节点测速自动选择、节点排除等功能，并附带web面板可供查看日志连接信息等。PaoPao GateWay配合PaoPaoDNS的`CUSTOM_FORWARD`功能就可以完成简单精巧的分流。   
详情：https://github.com/kkkgo/PaoPaoGateWay   

## 构建说明
`sliamb/paopaodns`Docker镜像由Github Actions自动构建本仓库代码构建推送，你可以在[Actions](https://github.com/kkkgo/PaoPaoDNS/actions)查看构建日志，或者自行下载源码进行构建，只需要执行docker build即可，或者可以fork仓库然后使用Actions进行自动构建。   

## 附录：使用到的程序
unbound：
- https://nlnetlabs.nl/projects/unbound/about/  
- https://www.nlnetlabs.nl/documentation/unbound/howto-optimise/
- https://unbound.docs.nlnetlabs.nl/en/latest/

redis: https://hub.docker.com/_/redis  
dnscrypt:
- https://github.com/DNSCrypt/dnscrypt-proxy   
- https://github.com/DNSCrypt/dnscrypt-resolvers
- https://dnscrypt.info/  

mosdns:
- https://github.com/kkkgo/mosdns

Country-only-cn-private.mmdb:
- https://github.com/kkkgo/Country-only-cn-private.mmdb
