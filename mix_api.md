### [English](./mix_api_en.md)

# 金本位合约交易API

* [概述](#open-api)

* [频率控制](#open-apilimited)

* [开启API权限](#open-apisecret)

* [代码示例](#open-apicode)

* [状态码](#open-apistatuscode)

* [金本位API列表](#open-apimixlist)
  * [获取交易对列表](#open-apimixlist-mixsymbollist)
  * [用户余额(持仓)](#open-apimixlist-mixaccountbalancelist)
  * [修改保证金](#open-apimixlist-mixaccounttransfermargin)
  * [下单](#open-apimixlist-mixorder)
  * [撤单](#open-apimixlist-mixremove)
  * [一键撤单](#open-apimixlist-mixremoveall)
  * [平仓](#open-apimixlist-mixclose)
  * [当前委托](#open-apimixlist-mixactiveorders)
  * [委托历史](#open-apimixlist-mixorderhistory)
  * [已成交](#open-apimixlist-mixaccountorderfills)

-----------

## <span id="open-api">概述 </span>

- 所有交易API请求都使用HTTP POST
- 交易API需要在官网申请API需要的key/secret
- 请求的header里添加version = '2.0'
- 请求的header里添加key/sign，sign=hash('sha256', $post_data.$secret)
- 请求的nonce参数为当前系统时间戳，单位为秒，nonce不早/晚于当前系统时间10秒
- 访问频率最快为100ms间隔
- 访问bjx站点的币对时,需要在请求的header里添加from = 'bjx'

## <span id="open-apisecret">频率控制 </span>  
我们对API的请求频率进行控制，具体频率参数请参考接口详情

对 API 的请求，以下标头将被返回︰
```
"X-ratelimit-limit: 1000"
"X-ratelimit-next: 500"
```
X-ratelimit-limit为当前接口的频率控制间隔,具体因接口不同而参数不同
如果你已经被频率限制，你将收到 403 响应, 以及一个额外的标头X-ratelimit-next, 它意味着你在重试前需要等待的时间
 
### 请求认证
在调用 API 时，需要提供 API Key 作为每个请求的身份识别，并且通过secret对请求数据加签

#### 公共参数

字段名 | 字段释义 | 字段类型 | 是否必填 | 默认值 | 参数类型 | 说明
 :-: | :-: | :-: | :-: | :-: | :-: | :-: 
key | 在平台申请的API_KEY |  string | 是 | 无 | http header | 用于身份识别
sign | 签名信息 |  string | 是 | 无 |  http header |按照一定规则形成的签名信息
version| api版本| string | 是 | 2.0 | http header | 用于区分api版本
nonce | 请求发起时的时间戳,单位:秒 | string | 是 | 无 |  http body | nonce不早/晚于当前系统时间10秒


 #### 如何进行签名
 1. 将对应业务的接口参数和除sign外的公共参数以http GET请求形式拼接, 如下
 
 ``` php
$post_data = 'leverage=100&symbol=BTCUSD&nonce=1542434791';

 ```
 
 2. 对拼接后的字符串进行加签
 ``` php
$secret = 't7T0YlFnYXk0Fx3JswQsDrViLg1Gh3DUU5Mr';
$sign = hash('sha256', $post_data.$secret)
 // sign = 670e3e4aa32b243f2dedf1dafcec2fd17a440e71b05681550416507de591d908
 ```
 
 3.header附加上key和sign参数，发送http请求
 
 ```http
 POST /order/active HTTP/1.1
 Content-Type: application/x-www-form-urlencoded
 key: 43f2dedf1dafcec2fd17a440e71b056815
 sign: 670e3e4aa32b243f2dedf1dafcec2fd17a440e71b05681550416507de591d908
 version: 2.0 
 
leverage=100&symbol=BTCUSD&nonce=1542434791
 
 ```

## <span id="open-apicode">代码示例 </span> 
Python：
``` Python
import requests
import hashlib
import json
import time
from urllib import parse

key = 'your key'
secret = 'your secret'
url = "https://api.bjx.io/order/active"
symbol = 'BTC_USDT'
nonce = int(time.time())
payload = {'nonce': nonce, 'symbol': symbol, 'page': 1, 'size': 10}
payload_str = parse.urlencode(payload)
sign = hashlib.sha256((payload_str+secret).encode("utf-8")).hexdigest()
headers = {'Content-Type': 'application/json', 'version': '2.0', 'key': key, 'sign': sign}
response = requests.post(url, data=json.dumps(payload), headers=headers)
```

PHP：
``` PHP
$key = 'your key'
$secret = 'your secret'
$url = "https://api.bjx.io/order/active";
$symbol = 'BTC_USDT';
$nonce = time();
$payload = ['nonce' => $nonce, 'symbol' => $symbol, 'page' => 1, 'size' => 10];
$payload_str = http_build_query($payload, '', '&');
$sign = hash('sha256', urldecode($payload_str).$secret);
$headers = ['Content-Type' => 'application/json', 'version'=> '2.0', 'key' => $key, 'sign' => $sign];
$response = Requests::post($url, $headers, json_encode($payload));
```

JavaScript：
``` JavaScript
let key = 'your key'
let secret = 'your secret'
let url = "https://api.bjx.io/order/active"
let symbol = 'BTC_USDT'
let nonce = Math.floor(Date.now() / 1000)
let sign = sha256("nonce="+ nonce + "&symbol=" + symbol + "&page=" + 1 + "&size=" + 10 + secret)
$.ajax({
  url: url,
  type: 'post',
  data: {
    nonce: nonce,
    symbol: symbol,
    page: 1,
    size: 10
  },
  headers: {
    version: '2.0',
    key: key,
    sign: sign
  },
  ...
```


## <span id="open-apistatuscode">状态码 </span>

| 错误代码        | 详细描述    |    
| :-----    | :-----   |    
|200	|	正常|    
|400	|	缺少参数|    
|401	|	缺少认证|    
|403	|	请求过快|    
|413	|	请求过大|    
|500	|	非法请求|    
|30001	|	交易对不存在|    
|30002	|	下单数量不合法|    
|30003	|	下单金额不合法| 

## <span id="open-apimixlist">金本位API列表 </span>

### <span id="open-apimixlist-mixsymbollist">获取交易对列表 POST /mix/symbol/list</span>
- 参数
 - site 站点 1:bjx
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  - id 唯一编号
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT),永续合约接口里传的currency参数和symbol参数都改成这个合约名name做为参数
  - symbol 交易对名字 例:MIX_ETH
  - product MIX
  - currency 币种(BTC,ETH,EOS)
  - amount_scale 数量精度
  - price_scale 价格精度
  - min_amount 最小下单数量
  - max_amount 最大下单数量
  - im 基础 IM
  - mm 基础 MM
  - make_rate maker费率
  - take_rate taker费率
  - state 1可用 0不可用
  - site 站点 1:bjx
  - multiplier 之前的乘数 现在用来表示单张合约价值
    - value_scale 价值精度
    - max_total 最大下单价值
    - min_total 最小下单价值
    - base_risk 基数风险限额
    - gap_risk 风险限额递增值
    - max_risk 最大风险额度
    - max_leverage 最大杠杆
    - leverages 所用杠杆
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/symbol/list -d 'nonce=1536826456'
```

	
### <span id="open-apimixlist-mixaccountbalancelist">用户余额(持仓) POST /mix/account/balance/list</span>
- 参数
  - user_id 用户id
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  - user_id 用户id
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)
  - available 资产净值(总余额)
  - available_balance 可用余额
  - currency 币种(ETH,EOS)
  - rate 汇率
  - side 1做多 2做空
  - holding 目前仓位数量
  - price 开仓价格
  - leverage 杠杆倍数
  - margin_user 用户设置的仓位保证金
  - margin_position 仓位保证金
  - margin_delegation 委托保证金
  - unrealized 未实现盈亏
  - realized 已实现盈亏
  - risk_limit 风险限额
  - liq_price 强平价格
  - mark_price 标记价格
  - adl ADL值
  - future_close_id 平仓委托ID
  - close_position_price 平仓委托价格
  - future_tp_id 止盈委托ID
  - future_sl_id 止损委托ID
  - tp_price 止盈委托价格
  - sl_price 止损委托价格
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/account/balance/list -d 'nonce=1536826456'
```
	
### <span id="open-apimixlist-mixaccounttransfermargin">修改保证金 POST /mix/account/transfer_margin</span>
- 参数
  - user_id 用户id
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)
  - side 1做多 2做空
  - amount 增加或减少数量
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data 详见 /mix/account/balance/list
  - message
- 字段说明
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/account/transfer_margin -d 'nonce=1536826456&name=BTCUSDT&side=1&amount=1'
```

## 订单相关
### <span id="open-apimixlist-mixorder">下单 POST /mix/order</span>
- 参数
  - user_id
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)
  - currency 币种(USDT,BTC,ETH,EOS)
  - side 订单类型 1开多(买入开多) 2开空(卖出开空) 3平多(卖出平多) 4平空(买入平空)
  - amount 数量必须大于0
  - price 价格必须大于0
  - type 下单类型 1 限价 2市价 3止盈止损
  - passive 是否被动委托 0否 1是
  - trigger_price 触发价格
  - trigger_type 触发类型 0默认 1盘口价格 2标记价格 3指数价格
  - tp_type 止盈触发类型 0默认 1盘口价格 2标记价格 3指数价格 如果是-1的话代表从仓位里下的触发单
  - tp_price 止盈价格
  - sl_type 止损触发类型 0默认 1盘口价格 2标记价格 3指数价格 如果是-1的话代表从仓位里下的触发单
  - sl_price 止损价格
  
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/order -d 'nonce=1536826456'
```

### <span id="open-apimixlist-mixremove">撤单 POST /mix/remove</span>
- 参数
  - user_id
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT) 
  - order_id 订单id 
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
- 限定访问间隔时间
	-	100毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/remove -d 'nonce=1536826456&name=BTCUSDT&order_id=1'
```

### <span id="open-apimixlist-mixremoveall">一键撤单 POST /mix/remove_all</span>
- 参数
  - user_id
  - trigger 0活动委托 1止损委托
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
- 限定访问间隔时间
	-	5000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/remove_all -d 'nonce=1536826456&trigger=1'
```

### <span id="open-apimixlist-mixclose">平仓 POST /mix/close</span>
- 参数
  - user_id
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)
  - side 1做多 2做空
  - price 平仓价格
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/close -d 'nonce=1536826456&name=BTCUSDT&side=1&price=13.00'
```

### <span id="open-apimixlist-mixactiveorders">当前委托 POST /mix/activeorders</span>
- 参数
  - user_id
  - page
  - size
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  - 订单id
    - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)
    - currency 币种(USDT,BTC,ETH,EOS)
    - amount 数量
    - price 委托价格
    - total 已成交额
    - executed 已成交量
    - state 状态 1委托中未成交 2委托中部分成交
    - create_time 下单时间
    - update_time 更新时间
    - tp_type 止盈触发类型 0默认 1盘口价格 2标记价格 3指数价格
    - tp_price 止盈价格
    - sl_type 止损触发类型 0默认 1盘口价格 2标记价格 3指数价格
    - sl_price 止损价格
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/activeorders -d 'nonce=1536826456&page=1&size=10'
```

### <span id="open-apimixlist-mixorderhistory">委托历史 POST /mix/orderhistory</span>
- 参数
  - user_id
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)（非必传）默认全部
  - page
  - size
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  - 订单id
    - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)
    - amount 数量(张)
    - price 委托价格
    - total 已成交额
    - executed 已成交量(张)
    - state 状态 1:委托中未成交, 2:委托中限价部分成交, 3:完全成交, 4:撤单全部, 5:撤单部分成交, 6:市价部分成交, 7:市价部分成交盘口全被吃空
    - create_time 下单时间
    - update_time 更新时间
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/orderhistory -d 'nonce=1536826456&name=BTCUSDT&page=1&size=10'
```

### <span id="open-apimixlist-mixaccountorderfills">已成交 POST /mix/account/orderfills</span>
- 参数
  - user_id
  - page
  - size
- 返回值
  - code(200表示正常读取data内容，非200则表示失败读取message失败信息)
  - data
  - message
- 字段说明
  - id 成交单ID
  - origin 1成交单,2强平单,3资金费率,4ADL减仓
  - order_id 订单ID
  - uid 用户ID
  - name 合约名(BTCUSDT,EHTUSDT,EOSUSDT)
  - side 方向 1开多(买入开多) 2开空(卖出开空) 3平多(卖出平多) 4平空(买入平空)
  - type 1限价 2市价 3止盈止损
  - price 成交价
  - amount 成交量
  - amount_total 委托数量
  - amount_last 成交量
  - total 成交额
  - fee 手续费
  - create_time 成交时间
- 限定访问间隔时间
	-	1000毫秒
```
curl -H 'key: xxx' -H 'sign: yyy' -H 'version: 2.0' -X POST https://api.bjx.io/mix/account/orderfills -d 'nonce=1536826456&page=1&size=10'
```
