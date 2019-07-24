
--------

###云桌面开发环境的信息

* 开发环境：
>   * 登陆地址：[https://160.10.0.40/pages/login.do?isLogout=1](https://160.10.0.40/pages/login.do?isLogout=1)
>   * 账号/密码：K0911216/Admin@123

gitlab账号：
>   * 地址：[http://gitlab.htzq.htsc.com.cn/ZhangleDsj/hedging/tree/master](http://gitlab.htzq.htsc.com.cn/ZhangleDsj/hedging/tree/master)
>   * 账号/密码：K0911216/Admin@123

* svn信息：
>   * 地址：[https://47.101.214.185/svn/file_trans](https://47.101.214.185/svn/file_trans)
>   * 账号/密码：muyan.dtwave/123qweasd

--------

###开发测试环境的数据源连接信息

####数据源测试环境信息(均在云桌面开发环境访问有效)

* 各数据库配置信息所在环境地址：
>   * [http://168.61.13.36:3000/?orgId=1](http://168.61.13.36:3000/?orgId=1) 
>   * 账号/密码：            admin/yinchao


Influxdb：13个指标

    1.封装的http接口地址：

>   * [http://168.63.66.120:6666](http://168.63.66.120:6666) 
    2.账号/密码：无需

OpenTSDB：5个指标

>   * 1.集群ip端口：168.61.9.49:4242,168.61.9.50:4242,168.61.9.51:4242
    
ElasticSearch：6个指标
    
>   * 在 *****【生产环境Grafana指标展示】***** 中查看数据源配置

Octopus 日志平台：9个指标

>   * 在 *****【生产环境Grafana指标展示】***** 中查看数据源配置
    
--------
###指标所在数据表信息梳理
####测试环境部分指标：
<i style="red">目前大多指标接口是来自Influxdb数据库</i>


* 策略数据
    * 在 default.xtrader.ats.stratgyStatistic 表
    * 查询条件 time、stratgyName 字段
    * 展示 count 字段
    
如：
```sql
A:select field(count) from default.xtrader.ats.stratgyStatistic where stratgyName = EasyStratgy FORMAT AS Time series Alias By easy 
B:select field(count) from default.xtrader.ats.stratgyStatistic where stratgyName = AlphaStratgy FORMAT AS Time series Alias By alpha

```

* 行情系统监控
    * 在 default.fast.quotamonitor.ztt  表
    * 查询条件 time 字段
    * 展示 ERROR、INFO、WARN 字段
    
如：
```sql

A:select field(ERROR) from default.fast.quotamonitor.ztt FORMAT AS Time series
B:select field(INFO) from default.fast.quotamonitor.ztt FORMAT AS Time series
C:select field(WARN) from default.fast.quotamonitor.ztt FORMAT AS Time series

```

* 策略去总收益(last 1h;amount 1h)
    * 在 default.t0.totalNetProfit 表
    * 查询条件 time 字段 
    * 展示 value 字段
    * 显示 258元
    
如：
```sql
A:select field(value) from default.t0.totalNetProfit FORMAT AS Time series 
```

* 策略总成交额(last 1h;amount 1h)
    * 在 default.t0.totalVolume 表
    * 查询条件 time 字段 
    * 展示 value/100000000 字段
    * 显示 0.03亿元

如：
```sql
A:select field(math(value/100000000)) from default.t0.totalVolume FORMAT AS Time series
```

* 策略总实例数
    * 在 default.xtrader.ats.strategyStatistic 表
    * 查询条件 time、name、strategyNum 字段 
    * 展示 userCount、count 字段
    * 显示 2.146K


如：
```sql

A:select field(userCount) from default.xtrader.ats.strategyStatistic where name = EasyStratgy FORMAT AS Time series 
-- N/A
B:select field(count) from default.xtrader.ats.strategyStatistic where strategyNum = strategyNum FORMAT AS Time series 
-- 2.146K
```

* 订单总数
    * 在 default.orderStatistic 表
    * 查询条件 time 字段 
    * 展示 T1 字段
    * 显示 -- N/A

如：
```sql

A:select field(count(T1)) from default.orderStatistic group by fill(0) FORMAT AS Time series 
-- N/A
```

* 策略总收益&成交额
    * 在 default.t0.strategy.netProfit 表
    * 查询条件 time、endpoint 字段 
    * 展示 T1 字段

如：
```sql

A:select field(value) from default.t0.strategy.netProfit where endpoint = EasyStratgy FORMAT AS Time series Alias by EasyStratgy总收益
B:select field(value) from default.t0.strategy.netProfit where endpoint = AlgoStratgy FORMAT AS Time series Alias by AlgoStratgy总收益
```


* Panel Title(datasource:zt-proxy)
    * 在 default.orderStatistic 表
    * 查询条件 time、clOrderId 字段 
    * 展示 T1, T2, T3, T4, T5, A2 字段

如：
```sql

A:select 
max(T1) alias T1,
max(T2) alias T2,
max(T3) alias T3,
max(T4) alias T4,
max(T5) alias T5,
max(A2) alias T2
from default.orderStatistic group by tag(clOrderId) format as table
```

* 日内策略总成交额
    * 在 default.t0.totalVolume 表
    * 查询条件 time、endpoint 字段 
    * 展示 value 字段
    * 显示 3207935元

如：
```sql

A:select field(value) from default.t0.totalVolume where endpoint = t0 FORMAT AS Time series
-- 3207935元
```

* 日内策略总收益
    * 在 t0.totalNetProfit 表
    * 查询条件 time 字段 
    * 展示 value 字段
    * 显示 -855元

如：
```sql
A:select value from t0.totalNetProfit FORMAT AS Time series
-- -855元
```

* 订单系统订单延时
    * 在 orderStatistic 表
    * 查询条件 time、endpoint 字段 
    * 计算 mean(T1，T1, T2, T3, T4, T5) 字段

如：
```sql

A:select mean("T1") FROM "orderStatistic" where ("endpoint" = 'uat-test' and $timefilter group by time(5m))
B:select mean("T2") FROM "orderStatistic" where ("endpoint" = 'uat-test' and $timefilter group by time(5m))
C:select mean("T3") FROM "orderStatistic" where ("endpoint" = 'uat-test' and $timefilter group by time(5m))
D:select mean("T4") FROM "orderStatistic" where ("endpoint" = 'uat-test' and $timefilter group by time(5m))
E:select mean("T5") FROM "orderStatistic" where ("endpoint" = 'uat-test' and $timefilter group by time(5m))
```

------

* 订单系统订单延时-深交所
    * 在 orderStatistic 表
    * 查询条件 time、clOrdId 字段 
    * 展示 "A2_A1","A3_A1","A4_A1" 字段

如：
```sql

select "A2_A1","A3_A1","A4_A1" from "orderStatistic" group by tag(clOrdId) alias by "深交所"
select "T4" from "orderStatistic" where ("endpoint" = 'all-test' and "exchange"="2") and $timeFilter
```

* 订单延时-深交所
    * 在 orderStatistic 表
    * 查询条件 time、clOrdId 字段 
    * 展示 "A2_A1","A3_A1","A4_A1" 字段

如：
```sql

select "A2_A1","A3_A1","A4_A1" from "orderStatistic" where $timeFilter group by tag(clOrderId) alias by "深交所"
```

* alpha调仓策略交易量
    * 在 t0.AlphaStrategyVolume  表
    * 查询条件 time、endpoint 字段 
    * 展示 "A2_A1","A3_A1","A4_A1" 字段

如：
```sql

A:select value from t0.AlphaStrategyVolume where endpoint = "t0"
```


* 算法集群内存负载监控
    * 在 mem.memused.percent 表
    * 查询条件 time、endpoint 字段 
    * 计算 mean("value") 字段

如：
```sql

A:select mean("value") from "mem.memused.percent" where (endpoint = "168.63.17.84") and group by time(1m) fill(null) FORMAT AS Time series Alias by oms-168.63.17.84
B:select mean("value") from "mem.memused.percent" where (endpoint = "168.63.32.78") and group by time(1m) fill(null) FORMAT AS Time series Alias by algo-168.63.32.78
C:select mean("value") from "mem.memused.percent" where (endpoint = "168.63.32.86") and group by time(1m) fill(null) FORMAT AS Time series Alias by algo-168.63.32.86
```

* cpu.busy
    * 在 cpu.busy 表
    * 查询条件 time、endpoint 字段 
    * 展示 "value" 字段

如：
```sql

A:select value from "cpu.busy" where ("endpoint" = "168.63.66.98") and $timefilter group by "endpoint" alias by transfer($tag_endpoint)
A:select value from "cpu.busy" where ("endpoint" = "168.63.66.99") and $timefilter group by "endpoint" alias by influx_proxy($tag_endpoint)
A:select value from "cpu.busy" where ("endpoint" = "168.63.66.100") and $timefilter group by "endpoint" alias by influxdb($tag_endpoint)
```


* memused.percent
    * 在 mem.memused.percent 表
    * 查询条件 time、endpoint 字段 
    * 展示 "value" 字段

如：
```sql

A:select "value" from "mem.memused.percent" where ("endpoint"="168.63.66.98") and $timeFilter group by "endpoint"
B:select "value" from "mem.memused.percent" where ("endpoint"="168.63.66.99") and $timeFilter group by "endpoint"
C:select "value" from "mem.memused.percent" where ("endpoint"="168.63.66.100") and $timeFilter group by "endpoint"
```

* network I/O
    * 在 net.if.in.bytes 表
    * 查询条件 time、endpoint 字段 
    * 计算 derivative("value",1s)  字段

如：
```sql

input-A:select derivative("value",1s) from "net.if.in.bytes" where ("endpoint" = "168.63.66.98") and $timeFilter group by "endpoint";
input-B:select derivative("value",1s) from "net.if.in.bytes" where ("endpoint" = "168.63.66.99") and $timeFilter group by "endpoint";
input-C:select derivative("value",1s) from "net.if.in.bytes" where ("endpoint" = "168.63.66.100") and $timeFilter group by "endpoint";
output-D:select derivative("value",1s) from "net.if.in.bytes" where ("endpoint" = "168.63.66.98") and $timeFilter group by "endpoint";
output-E:select derivative("value",1s) from "net.if.in.bytes" where ("endpoint" = "168.63.66.99") and $timeFilter group by "endpoint";
output-F:select derivative("value",1s) from "net.if.in.bytes" where ("endpoint" = "168.63.66.100") and $timeFilter group by "endpoint";
```

* disk.usage
    * 在 df.bytes.used.percent 表
    * 查询条件 time、endpoint 字段 
    * 展示 value  字段

如：
```sql

A:select "value" from "df.bytes.used.percent" where ("endpoint" = "168.63.66.98" and "mount" = "/app") and $timeFilter group by "endpoint","mount"
B:select "value" from "df.bytes.used.percent" where ("endpoint" = "168.63.66.99" and "mount" = "/app") and $timeFilter group by "endpoint","mount"
C:select "value" from "df.bytes.used.percent" where ("endpoint" = "168.63.66.100" and "mount" = "/app") and $timeFilter group by "endpoint","mount"
```


--------

####DEMO

```
http request:

GET:http://168.63.66.120:6666/query

headers:{
    "Content-type":"application/x-www-form-urlencoded"
}

body:{
    "db":"test",
    "q":"select value from cpu.busy" where (endpoint = '168.63.66.98') AND time >= now() -15m GROUP BY 'endpoint';
    select value from cpu.busy" where (endpoint = '168.63.66.99') AND time >= now() -15m GROUP BY 'endpoint';
    select value from cpu.busy" where (endpoint = '168.63.66.100') AND time >= now() -15m GROUP BY 'endpoint';"
}
    

```

```json 
http resopnse
{
    "result":[
        {   
            "statement_id":0,
            "series":[...]
        }
    ]
}
```
