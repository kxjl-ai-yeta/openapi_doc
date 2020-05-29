# yeta开放平台开发接口

----------


### 目录 
Part1：平台能力调用
应用方请求平台，调用对应能力。

Part2：结果数据推送
平台向应用方推送结果数据。

Part3：业务能力对接
平台调用应用方能力，拉取机器人交互数据。



# Part1: 平台能力调用

----------


### 接口协议说明

请求方向：由应用方主动请求yeta开放平台

请求方式：HTTP POST

字符编码：UTF-8

请求头：Content-Type: application/json;charset=UTF-8

返回格式：json

#### 返回json对象说明

yeta开放平台的所有json响应消息采用标准对象定义，全部具有code、message和result三个属性。

名称|类型|说明 
---|:-:|:-: 
code|int|返回码。0：成功，其它值表示失败，具体参考“附录：返回码参照”
message| string|返回码描述
result|object|返回结果集。

----------


# 一、获取token
功能：获取授权令牌。请求本接口之外的其它接口时，必须在接口url中通过token参数携带令牌。令牌有效期默认1小时，需要在过期前重新获取。

接口地址：https://www.xfyeta.com/openapi/oauth/v1/token  

## 请求参数说明
名称|必填|类型| 说明
---|:-:|:-:|:-:
app_key|是|string|应用的app_key
app_secret|是|string|应用的app_secret
### 请求示例
~~~
POST /openapi/oauth/v1/token HTTP/1.1
Host: www.xfyeta.com
Content-Type: application/json;charset=UTF-8
Cache-Control: no-cache

{
   "app_key": "ODM1ZTk4ODAtYTMyZC00ZjBiLTkzMDQtY2VjNWU0ZDUyZWQ5",
   "app_secret": "MTM5NUM3NjlGQ0M2REUwN0FBREE3QjUxMkU1Qzg5NUQ="
}
~~~
### 响应示例
~~~
HTTP/1.1 200 OK
Content-Length:98
Date:Fri, 10 Aug 2018 06:58:01 GMT

{
    "code": 0,  
    "message": "ok",  
    "result": {
       "token": "08236d0aeeee4d5b566db5f4adc41a63",
       "time_expire": 3600
    }      
}
~~~

### result返回结果集说明
名称|必填| 类型|说明|备注
---|:-:|:-:|:-:|:-:
token|是|string|令牌|
time_expire|是|long|有效期|单位：秒。默认3600。


----------


# 二、查询配置
功能：查询企业下的各项资源及配置信息

接口地址：https://www.xfyeta.com/openapi/config/v1/query?token=08236d0aeeee4d5b566db5f4adc41a63  


## 请求参数说明
名称|必填|类型|说明
---|:-:|:-:|:-:
type|否|int|配置项分类。1：话术，2：线路，3：接口，4：发音人，0：全部（默认）。

### 请求示例
~~~
{
   "type": 0
}
~~~

### 响应示例
~~~
{
    "code": 0,  
    "message": "ok",  
    "result": {
       "lines": [
           {
               "line_num": "055169101407",
               "concurrents": 10,
               "time_work": ["09:00:00-12:00:00", "13:00:00-17:59:59"],
               "status": 1,
               "time_apply": 1527321492000,
               "time_expire": 2592000
           },
           {
               "line_num": "055169101408",
               "concurrents": 100,
               "time_work": ["08:00:00-20:00:00"],
               "status": 0,
               "time_apply": 1527321492000,
               "time_expire": 2592000
           }
       ],
       "robots": [
           {
               "robot_id": "111111",
               "robot_name": "金融",
               "call_column":["客户手机号码","姓名","性别"],
               "status": 4,
               "type": 1,
               "deleted": 0,
               "time_create": 1527321492000,
               "time_update": 1527325092000
           }
       ],
       "urls": [
           {
               "url": "http://your-service.url/receiveCallRecord",
               "url_module": "receiveCallRecord"
           }
       ],
       "voices": [
           {
               "voice_code": "60020",
               "voice_name": "春春"
           }
       ]
    }  
    
}  
~~~


## result返回结果集说明
名称| 类型|说明|备注
---|:-:|:-:|:-:
**lines**|object[]|线路|
line_num|string|号码|
concurrents|int|并发数|
time_work|string[]|工作时段|
status|int|状态|0：空闲，1：任务占用中。
time_apply|long|申请时间|毫秒时间戳
time_expire|long|有效期|单位：秒。-1：永久有效。
**robots**|object[]|话术|
robot_id|string|话术编号|
robot_name|string|话术名称|
call_column|string[]|外呼数据列模板|第一列必须是客户手机号，其他列是话术动态信息。
status|int|话术状态|1：审核中，2：未通过，3：待发布，4：已发布。
type|int|话术类型|1：普通话术，2：动态话术。
deleted|int|删除标记|0：未删除，1：已删除。
time_create|long|创建时间|毫秒时间戳
time_update|long|更新时间|毫秒时间戳
**urls**|object[]|接口配置|
url|string|接口URL|应用方提供给yeta平台调用的接口服务url地址
url_module|string|接口模块|详见Part2和Part3。
**voices**|object[]|发音人|
voice_code|string|发音人编码
voice_name|string|发音人名称

### 常用发音人清单
voice_name|voice_code|性别
---|:-:|:-:
春春	|65580|女声
晓诗	|62140|女声
晓燕	|60020|女声
晓峰	|60030|男声
小陈	|65660|男声
楠楠	|60130|女声
晓医	|65600|女声

具体发音效果建议通过Web话术预览进行体验。

----------


# 三、直接外呼
功能：面向便捷外呼业务场景，提交号码数据同时指定线路和机器人话术，直接发起外呼。直接外呼的号码数据将被提交到应用默认对应的长期任务下。如果号码数据中存在不合规记录（例如敏感号码、非手机号等）将会整批失败。


接口地址：https://www.xfyeta.com/openapi/outbound/v1/task/callout?token=08236d0aeeee4d5b566db5f4adc41a63  


## 请求参数说明
名称| 必填|类型|说明|备注
---|:-:|:-:|:-:|:-:
robot_id|是|string|话术编号|
line_num|是|string|线路号码|
call_column|是|string[]|外呼数据列| 
call_list|是|string[][]|外呼数据行|单次上限50条
voice_code|否|string|发音人编码|
robot_speed|否|number|发音人语速|取值范围[-500,500],0为原速，数值大则语速快，对应于0.5~1.5倍线性关系
### 请求示例
~~~
{
    "line_num":"69101338",
    "robot_id":"719",
    "call_column":["客户手机号码","姓名"],
    "call_list":[["19900000001","张先生"],["19900000002","王女士"]],
    "voice_code":"60030",
    "robot_speed": 200
}

~~~
### 响应示例
~~~
{
    "code": 0,  
    "message": "ok",  
    "result": {
        "total": 2,
        "task_data_ids": [130,131]
    }     
}
~~~ 

## result返回结果集说明
名称| 类型|说明
---|:-:|:-:
total|int|号码总数
task_data_ids|long[]|外呼数据行对应的任务数据编号，用于结果推送数据关联。


----------


# 四、创建外呼任务
功能：面向需要灵活管控的业务场景。可以按照不同的业务维度创建多组任务、分批多次向指定任务提交号码数据，可以对外呼任务进行启动、暂停、删除等控制操作。

接口地址：https://www.xfyeta.com/openapi/outbound/v1/task/create?token=08236d0aeeee4d5b566db5f4adc41a63  


## 请求参数说明
名称| 必填|类型|说明|备注
---|:-:|:-:|:-:|:-:
task_name|是|string|任务名称|
line_num|是|string|线路号码|如果是多个，分号分隔
robot_id|是|string|话术id|
time_range|否|string[]|外呼时间段|
time_begin|是|long|任务开始时间|毫秒时间戳
time_end|否|long|任务结束时间|毫秒时间戳
voice_code|否|string|发音人编码|
trubo_mode|否|boolean|高性能模式|默认false

### 请求示例
~~~
{
   "task_name": "测试外呼",
   "line_num": "055169101407",
   "robot_id": "11",
   "time_range": ["09:00:00-12:00:00", "13:00:00-17:30:00"],
   "time_begin": 1527321492000,
   "time_end": 1527325092000
}
~~~

### 响应示例

~~~
{
    "code": 0,  
    "message": "ok",  
    "result": {
       "task_id": "129"
    }     
}
~~~ 


## result返回结果集说明
名称| 类型|说明
---|:-:|:-:
task_id|string|任务Id。用于任务数据提交和管理。


----------


# 五、提交任务数据
功能：向指定任务提交号码数据，可以分多批次提交。

接口地址：https://www.xfyeta.com/openapi/outbound/v1/task/insert?token=08236d0aeeee4d5b566db5f4adc41a63  


## 请求参数说明
名称| 必填|类型|说明|备注
---|:-:|:-:|:-:|:-:
task_id|是|string|任务id|
call_column|是|string[]|数据列映射|
call_list|是|string[][]|数据行|单次上限50条

### 请求示例
~~~
{
   "task_id": "129",
   "call_column": ["客户手机号码","列2", "列3"],
   "call_list": [
         ["19900000001","t2","t3"],
         ["19900000002","t2","t3"]
     ]
}
~~~  

### 响应示例

~~~
{
    "code": 0,  
    "message": "ok",  
    "result": {
       "total": 2,
       "task_data_ids": [87,88]
    }  
}
~~~

## result返回结果集说明
名称| 类型|说明
---|:-:|:-:
total|int|号码总数
task_data_ids|long[]|外呼数据行对应的任务数据编号，用于结果推送数据关联。



----------

# 六、启动外呼任务
功能：启动外呼任务，任务将按照预设的开始时间和工作时段进行外呼。 任务启动之后，将不能再提交号码数据。

接口地址：https://www.xfyeta.com/openapi/outbound/v1/task/start?token=08236d0aeeee4d5b566db5f4adc41a63   


## 请求参数说明
名称| 必填|类型|说明
---|:-:|:-:|:-:
task_id|是|string|任务id

### 请求示例
~~~
{
   "task_id": "129"
}
~~~

### 响应示例

~~~
{
    "code": 0,  
    "message": "ok",  
    "result": {}     
}
~~~ 

## result返回结果集说明
无。通过code响应码表示是否成功。


# 七、暂停外呼任务
功能：暂时停止任务呼叫。可以通过启动外呼任务接口恢复任务呼叫。

接口地址：https://www.xfyeta.com/openapi/outbound/v1/task/pause?token=08236d0aeeee4d5b566db5f4adc41a63   

## 请求参数说明
名称| 必填|类型|说明
---|:-:|:-:|:-:
task_id|是|string|任务id

### 请求示例
~~~
{
   "task_id": "129"
}
~~~

### 响应示例

~~~
{
    "code": 0,  
    "message": "ok",  
    "result": {}     
}
~~~ 

## result返回结果集说明
无。通过code响应码表示是否成功。

----------


# 八、删除外呼任务
功能：对外呼任务进行强制停止并删除，删除后不能再次启动。
接口地址：https://www.xfyeta.com/openapi/outbound/v1/task/delete?token=08236d0aeeee4d5b566db5f4adc41a63   


## 请求参数说明
名称| 必填|类型|说明
---|:-:|:-:|:-:
task_id|是|string|任务id

## 请求示例
~~~
{
   "task_id": "129"
}
~~~
### 响应示例

~~~
{
    "code": 0,  
    "message": "ok",  
    "result": {}     
}
~~~ 

## result返回结果集说明
无。通过code响应码表示是否成功。



# 九、查询任务
功能：查询任务信息和任务列表。

接口地址：https://www.xfyeta.com/openapi/outbound/v1/task/query?token=08236d0aeeee4d5b566db5f4adc41a63  

## 请求参数说明
名称| 必填|类型|说明|备注
---|:-:|:-:|:-:|:-:
task_id|否|string|任务id
time_begin|否|long|开始时间|
time_end|否|long|结束时间|
task_name|否|string|任务名称|模糊检索
task_status_list|否|int[]|任务状态数组|0：新建，1：启动，2：运行，3：手动暂停，4：完成，5：等待下轮重试，6：不在工作时段，7：过期
page_size|否|int|页大小|最大值50，默认20
page_index|否|int|当前页码|从1开始
sort_name|否|string|排序字段|ID：任务编号，NAME：任务名称，CREATETIME：任务创建时间，STARTTIME：任务开始时间，ENDTIME：任务结束时间
sort_order|否|string|排序字段方式|"ASC" 正序 "DESC" 倒序


### 请求示例
~~~
{
  "time_begin": 153000000
}
~~~

### 响应示例

~~~
{
    "code":0,
    "message":"ok",
    "result":{
        "total_rows":30,
        "rows":[
            {
                "task_id":"1909",
                "task_name":"task_test1",
                "status":4,
                "deleted":0,
                "time_task_start":1533289491345,
                "time_task_finish":1533820104848,
                "count_total_task":4,
                "count_tel":4,
                "count_recalled":0,
                "time_task_estimate_begin":1478157571217,
                "time_task_estimate_end":0,
                "line_num":"96866001",
                "robot_id":"719",
                "robot_name":"测试话术无参数",
                "voice_code":"60030",
                "voice_speed":1,
                "count_max_recall":2,
                "status_recall":"[1,1]",
                "time_recall_wait":2000,
                "time_range":"["07:30:00-23:30:00"]",
                "intention_push":"["A","B"]",
                "process_count":7,
                "process_tel_count":7,
                "process_through_count":4,
                "process_through_rate":1
            }
        ]
    }
}
~~~ 


## result返回结果集说明

名称|类型|必填|说明|值域
---|:-:|:-:|:-:|:-:
total_rows|int|是|总行数
**rows**|object[]|是|结果列表
task_id|string|是|任务id
task_name|string|否|任务名称
status|int|是|任务状态|0：新建，1：启动，2：运行，3：手动暂停，4：完成，5：等待下轮重试，6：不在工作时段，7：过期
task_type|int|是|任务类型|0：普通任务,2：任务池
deleted|int|否|删除标志|1：删除，0：正常
time_task_start|long|否|运行开始时间|毫秒时间戳
time_task_finish|long|否(仅限普通任务)|运行结束时间|毫秒时间戳
process_count|int|否|已呼叫次数
process_tel_count|int|否|已呼叫号码数
process_through_count|int|否|已接通量
process_through_rate|double|否|当前接通率
count_tel|int|否|任务号码量
count_recalled|int|否(仅限普通任务)|任务已重试次数
time_task_estimate_begin|long|否(仅限普通任务)|预设开始时间
time_task_estimate_end|long|否(仅限普通任务)|预设结束时间
line_num|string|否(仅限普通任务)|线路号码
robot_id|string|否(仅限普通任务)|话术id
robot_name|string|否(仅限普通任务)|话术名称
voice_code|string|否(仅限普通任务)|发音人编码
voice_speed|int|否(仅限普通任务)|发音人语速，默认1
count_max_recall|int|否(仅限普通任务)|预设任务重试次数
time_recall_wait|int|否(仅限普通任务)|预设重试外呼等待时间|单位：秒。
time_range|string|否(仅限普通任务)|预设外呼时间段
intention_push|string|否(仅限普通任务)|预设推送意向度门限

# 十、查询推送失败记录
功能：查询推送失败记录文件清单，提供7日内失败记录文件下载，不含当日。

接口地址：https://www.xfyeta.com/openapi/download/v1/push/failed?token=08236d0aeeee4d5b566db5f4adc41a63  

## 请求参数说明
名称| 必填|类型|说明|备注
---|:-:|:-:|:-:|:-:
date|否|string|日期|格式如yyyy-MM-dd

### 请求示例
~~~
{
  "date":"2018-12-10"
}
~~~  

### 响应示例

~~~
{
    "code": 0,
    "message": "ok",
    "result": {
        "url": [
            "4267/2018-12-10/0806090407040D0F0F020F060B080E09.zip"
        ]
    }
}
~~~

## result返回结果集说明
名称| 类型|说明
---|:-:|:-:
url|string[]|文件下载相对路径。URL前缀https://www.xfyeta.com/push_download/

### 文件示例

下载路径 https://www.xfyeta.com/push_download/150/2018-12-04/0E050E000F0B000A060D0D0C0B0B0B03.zip

提醒：zip文件中可能包含多个文件或多层文件夹，建议使用org.apache.commons.io.FileUtils进行遍历处理。

~~~

{"data":{"call_uuid":"01M21S470GDVR68B345H7B5AES01SKVK","business_id":"150","robot_id":"1161","task_id":"88785","task_data_id":"88547143","caller":"055196866001","callee":"19900000001","direction":2,"time_dial":1543935195000,"time_answer":1543935205000,"time_hangup":1543935211000,"duration_ring":10,"duration_call":6,"task_result":"15","task_result_desc":"客户接听后并主动挂机","task_tries":0,"task_row_index":3,"task_row_head":["客户手机号码"],"task_row_value":["18656957753"]},"module":"receiveCallRecord","retryTimes":4,"timestamp":1543935214693}
{"data":{"call_uuid":"8edf0c99-5b3d-4722-b1fb-10b2fbfecda2","business_id":"150","robot_id":"20001003","task_id":"504138886481984","task_data_id":"32264888839704576","caller":"055196866001","callee":"19900000001","direction":2,"time_dial":1543935305290,"time_answer":1543935313230,"time_hangup":1543935444490,"duration_ring":7,"duration_call":131,"task_result":"0","task_result_desc":"成功","task_tries":0,"task_row_index":1,"task_row_head":["客户手机号码"],"task_row_value":["18656957753"]},"module":"receiveCallRecord","retryTimes":4,"timestamp":1543935447903}

~~~
名称| 类型|说明
---|:-:|:-:
data|object|推送数据，数据结构与对应推送接口完全一致
module|string|模块名称，与Part2定义的消息名称一致
retryTimes|int|尝试次数
timestamp|long|消息创建时间戳



----------


# 附录：返回码参照
## 成功码参照
返回码|说明  
---|:-:
0 | ok（成功）
## 服务级错误码参照  
错误码|说明  
---|:-- 
200101|账号或密码错误

## 系统级错误码参照
错误码|说明  
---|:--  
10001|无效的token请求  
10002|该token无请求权限
10003|错误的app_id
10004|服务忙，请稍后再试
10005|输入参数不正确
10020|接口维护
10021|接口停用
200200|错误的数据关系
200201|错误的企业id
200202|错误的任务id
200203|错误的外呼号码
200204|错误的话术id
200205|错误的话术发音id
200206|错误的语速
200209|余额不足  
200210|错误的文件格式
200211|文件上传超过限定大小10m
200212|错误的数据，与话术动态信息不匹配
200213|上传的外呼数据量超过限制
200222|非有效工作时间段
3030002|号码被占用
3010102|任务已经删除
3010101|任务已经完成

## 错误码格式说明（示例：200201）
开头|中间| 尾部|
:--|:--|:--
2|002|01
服务级错误（1为系统级错误）|服务模块代码|具体错误代码



----------


# Part2:数据推送



### 接口协议说明

请求方向：由yeta开放平台主动请求应用方

请求方式：HTTP POST

字符编码：UTF-8

返回格式：text

对接说明：

应用方应当按照yeta开放平台的数据推送接口协议，实现并提供各类数据接收服务的接口URL地址；
yeta开放平台将会在每通电话结束后，主动向应用方URL进行HTTP POST话单、对话、录音的json数据；
应用方应当立即进行响应，响应内容不限格式，但必须包含success关键字。

强烈建议应用方在接收推送数据时采用异步处理机制，接收到推送数据后立即缓存数据并响应，
然后再使用异步服务对数据进行业务处理，避免因为网络传输、超时等原因造成推送数据丢失。

平台向应用方推送数据失败时（未能在3秒内接收到应用方返回的success关键字），平台将会重试3次，
对于重试后仍然无法推送的数据将被记录到文件中，应用方可在次日查询下载推送失败的数据。

业务数据关联关系：
外呼结果数据与任务数据通过task_data_id关联，呼入结果数据与通话通过call_relation_id关。

----------

# 一、话单推送

接口模块名称：receiveCallRecord

接口地址：[企业预先配置url]  

注意：为了满足应用方对即时性的要求，平台对外呼的话单可能会进行两次推送。两次推送分别为基本属性话单消息和外呼属性话单消息，外呼话单消息中包含基本话单消息的所有字段项，两种消息可以通过是否包含task_row_index属性进行区分。受限于网络等因素，两次话单推送的顺序不做保证。
可以通过接听时间戳time_answer判定是否接通。

## 请求参数说明
名称| 必填| 类型| 说明| 备注
---|:-:|:-:|:-:|:-:
business_id||string|企业id|
app_id||string|应用id|
robot_id||string|话术话术id|
call_uuid|是|string|通话UUID|录音和会话的唯一关联。
caller||string|主叫号码|
callee||string|被叫号码|
direction|是|int|呼叫方向|1:呼入 2 呼出
time_answer|是|long|接听时间戳|unix_time
time_hangup|是|long|挂断时间戳|unix_time
duration_ring|是|int|拨号振铃时长|秒
duration_call|是|int|通话时长|秒
call_relation_id|仅呼入|string|关联ID|业务侧的唯一标识 
task_id|仅呼出|string|外呼任务id|
task_data_id|仅呼出|string|外呼任务id|
time_dial|仅呼出|long|拨号时间戳|unix_time
time_ring|仅呼出|long|振铃时间戳|unix_time
task_result|仅呼出|string|外呼结果|
task_result_desc|仅呼出|string|外呼结果描述|
task_tries|仅呼出|int|外呼重试次数|
task_row_index|仅呼出|int|外呼数据行号|
task_row_head|仅呼出|array|外呼数据行头|
task_row_value|仅呼出|array|外呼数据行值|

#### 外呼结果说明
task_result| task_result_desc| 备注
---|:-:|:-:
-1 |超时|
0  |成功|
2  |正在通话中|
3  |无法接通|  无法接听,无法接通
4  |关机|
5  |用户正忙|  短忙音,长忙音,静音
6  |空号  |空号,改号
7  |号码错误|
8  |用户拒接|  用户拒接
9  |停机  |停机,暂停服务,欠费
15 |客户接听后并主动挂机|
16 |客户掉线|
18 |接通|
19 |转人工|
20 |用户未接|  回铃音,彩铃,无人接听,用户拒接,来电提醒
22 |来电提醒|
23 |呼入限制|
25 |呼叫转移|  呼叫转移失败
26 |网络故障|
27 |呼出受限 | 拨号方式不正确
28 |线路故障 | 线路故障,网络忙


### 请求示例
~~~
{
    "business_id":"2745",
    "app_id":"test",
    "robot_id":"2745001",
    "call_uuid":"123425c-6483-11e8-b8fa-2769c24918cd",
    "call_relation_id":"test-123",
    "task_id":"0",
    "task_data_id":"262381",
    "caller":"055196866001",
    "callee":"19900000001",
    "direction":1,
    "time_dial":1527737756729,
    "time_ring":1527737757449,
    "time_answer":1527737763429,
    "time_hangup":1527737784829,
    "duration_ring":7,
    "duration_call":28
}

~~~

### 响应结果示例
~~~
success
~~~


# 二、录音推送
接口模块名称：receiveVoice  
接口地址：[企业预先配置url]  


## 请求参数说明
名称| 必填| 类型| 说明| 备注
---|:-:|:-:|:-:|:-:
business_id||string|企业id|
app_id||string|应用id|
robot_id||string|话术id|
task_data_id||string|外呼任务id|
call_relation_id||string|呼入关联id|
call_uuid|是|string|通话UUID|
url|是|string|url路径|http://voice.kxjlcc.com:9000/
size|是|long|文件大小|字节
duration|是|int|录音长度|秒


### 请求示例
~~~

{
    "call_uuid":"809c825c-6483-11e8-b8db-2769c24918cd",
    "url":"ant/4/ivr/0/2018/05/31/0/452263-7571100e-7afc-42aa-9d69-ac25805d45b6.wav",
    "size":1043244,
    "duration":32,
    "business_id":"10527",
    "app_id":"123",
    "robot_id":"10527"
}

~~~

### 响应结果示例
~~~
success
~~~



# 三、会话推送

接口模块名称：receiveDialog  
接口地址：[企业预先配置url]  


## 请求参数说明
名称| 必填| 类型| 说明| 备注
---|:-:|:-:|:-:|:-:
business_id||string|企业id|
app_id||string|应用id|
robot_id||string|话术话术id|
task_data_id||string|外呼任务id|
call_relation_id||string|呼入关联id|
call_uuid|是|string|通话UUID|
tag_max||string|最高意向度|
tag_min||string|最低意向度|
time_begin||long|开始时间戳|
time_end||long|结束时间戳|
**dialog**|是|array|交互记录|
seq||int|节点序号|
role||string|节点角色|robot话术customer客户
node_id||string|节点id|
node_name||string|节点名称|
node_type||string|节点类型|
label||string|节点标签|
tag_name||string|意向|
tag_desc||string|意向说明|
text_robot||string|话术输出内容|
text_man||string|用户输入内容|
question_id||String|问题uuid|
time_start||long|节点开始时间戳|
time_speaking||long|用户说话时间戳|
**hits**||array|命中|
hit||string|命中内容|
pick||boolean|是否选中|
answer_id||string|回答uuid|

#### node_type节点类型说明

node_type| 说明
---|:-:
AnyNode | 任意回答节点
CollectInputNode | 自定义输入节点
ConditionNode | 按键输入节点
EndNode | 结束节点
HookInfoNode | 动态引用节点
KeyNavigationNode | 按键导航节点
NoNeedAnswerNode | 无需回答节点
NormalNode | 普通交互节点



### 请求示例
~~~

{
    "business_id":"10527",
    "app_id":"123",
    "robot_id":"10527",
    "call_uuid":"f3b13ebc-6394-11e8-907d-ebcc4d560c04",
    "dialog":[
        {
            "seq":"3",
            "role":"robot",
            "node_id":"word_node_70601",
            "node_name":"开头语",
            "text_robot":"喂，您好！",
            "text_man":"",
            "question_id":""
        },
        {
            "seq":"4",
            "role":"customer",
            "node_id":"word_node_70601",
            "node_name":"开头语",
            "text_robot":"喂，您好！",
            "text_man":"",
            "question_id":"",
            "hits":[
                {
                    "hit":"任意回答",
                    "pick":true,
                    "answer_id":""
                }
            ]
        }
    ]
}


~~~

### 响应结果示例
~~~
success
~~~



----------


# Part3:业务能力对接

### 接口协议说明

请求方向：由yeta开放平台根据预设配置，按需请求应用方业务能力，拉取机器人运行时的业务数据

请求方式：HTTP POST

字符编码：UTF-8

返回格式：json

对接说明：

应用方应当按照yeta开放平台的接口协议，实现并提供此服务的接口URL地址；
yeta开放平台将会在每通电话开始时，主动向应用方URL进行HTTP POST请求，
获取此通电话对应的话术动态数据并执行。

强烈建议应用方的接口响应时间建议不超过500毫秒，且具备高可用性，避免影响客户电话交互体验。


----------

# 一、呼入话术上下文动态数据获取

**此接口仅面向电话呼入对接，且话术存在动态数据的业务场景，其它场景可以不实现此接口。**

yeta开放平台在电话呼入时主动请求应用方接口

接口模块名称：getDialogContext  
接口地址：[企业预先配置url]  


## 请求参数说明
名称| 必填| 类型| 说明| 备注
---|:-:|:-:|:-:|:-:
business_id||string|企业id|取自sip头X-business-id
app_id||string|应用id|取自sip头X-app-id
robot_id||string|话术话术id|取自sip头X-robot-id
call_relation_id|是|string|业务侧关联id|取自sip头X-call-relation-id



## 请求示例
~~~

{
	"call_relation_id":"1111",
	"business_id":"167",
	"app_id":"3333",
	"robot_id":"4444"
}

~~~
## 响应结果说明
map<string,string> 当前通话话术话术要求的上下文动态数据

## 响应结果示例
~~~
{
"key1": "value1",
"性别": "男",
"姓名": "张三"
}
~~~


# 二、语义接口
获取语义接口用于支持应用方的个性化语义处理需求场景；Yeta开放平台向应用方提供此次对话的基本属性、话术上下文动态数据和人机交互过程数据，应用方返回语义分析处理后的结果数据。

示例场景 —— 高血压语义接口：机器人询问用户血压之后，将用户语音输入内容提交给应用方，应用方根据其业务模型对输入的模糊内容进行语义分析和处理，提供评估诊断精确的结果数据。

#### node_type节点类型说明

node_type| 说明
---|:-:
AnyNode | 任意回答节点
CollectInputNode | 自定义输入节点
ConditionNode | 按键输入节点
EndNode | 结束节点
HookInfoNode | 动态引用节点
KeyNavigationNode | 按键导航节点
NoNeedAnswerNode | 无需回答节点
NormalNode | 普通交互节点
## 请求参数说明
名称| 必填| 类型| 说明| 备注
---|:-:|:-:|:-:|:-:
business_id||string|企业id|
app_id||string|应用id|
robot_id||string|话术话术id|
task_data_id||string|外呼任务id|
call_relation_id||string|呼入关联id|
call_uuid|是|string|通话UUID|
caller||string|主叫号码|
callee||string|被叫号码|
direction||string|呼叫方向1:呼入  2:呼出|
context||string|话术上下文动态数据|
code||string|接口功能代码	|
task_id|是|string|任务id|
**dialog**|是|array|交互记录|
seq||int|节点序号|
role||string|节点角色|robot话术customer客户
node_id||string|节点id|
node_name||string|节点名称|
node_type||string|节点类型|
label||string|节点标签|
tag_name||string|意向|
tag_desc||string|意向说明|
text_robot||string|话术输出内容|
text_man||string|用户输入内容|
question_id||String|问题uuid|
time_start||long|节点开始时间戳|
time_speaking||long|用户说话时间戳|
**hits**||array|命中|
hit||string|命中内容|
pick||boolean|是否选中|
answer_id||string|回答uuid|
## 请求示例
~~~
{
    "code": "SELF1",
    "task_data_id": 275821623314507,
    "callee": "19900000001",
    "dialog": [
        {
            "node_type": "NormalNode",
            "text_robot": "测试18805512125发短信功能",
            "tag_name": "B",
            "reject": 0,
            "tag_desc": "中zhong",
            "node_name": "开始节点",
            "text_man": "18005699514",
            "has_reply": 1,
            "un_hit": 1,
            "seq": 0,
            "node_id": "words_start_8fafbbb1-9"
        },
        {
            "node_type": "NormalNode",
            "text_robot": "测试发短信功能辅助",
            "tag_name": "B",
            "reject": 1,
            "tag_desc": "中zhong",
            "node_name": "开始节点",
            "text_man": "18005699514",
            "has_reply": 1,
            "un_hit": 2,
            "seq": 1,
            "node_id": "words_start_8fafbbb1-9"
        },
        {
            "hit": {
                "pick": true,
                "name": "多次未识别发短信回答",
                "id": "judge_node_d2691c8b-5"
            },
            "node_type": "NormalNode",
            "text_robot": "多次未识别发短信",
            "reject": 0,
            "node_name": "多次未识别发短信",
            "text_man": "我199123456789",
            "has_reply": 1,
            "un_hit": 0,
            "seq": 2,
            "node_id": "words_node_ebf2b05d-6"
        },
        {
            "hit": {
                "pick": true,
                "name": "新建按键正确回答",
                "id": "judge_node_7d4db64c-4"
            },
            "node_type": "ConditionNode",
            "text_robot": "这是按键话术",
            "reject": 0,
            "node_name": "按键话术",
            "text_man": "18005699514",
            "has_reply": 1,
            "un_hit": 0,
            "seq": 3,
            "node_id": "words_node_0997e480-6"
        },
        {
            "node_type": "NormalNode",
            "text_robot": "结束语发短信",
            "reject": 0,
            "node_name": "结束语发短信",
            "text_man": "18005699514",
            "has_reply": 1,
            "un_hit": 0,
            "seq": 4,
            "node_id": "words_start_07deddaa-1"
        }
    ],
    "caller": "055196866001",
    "robot_id": 1993,
    "context": {
        "jwyuan": "ryhan",
        "按键话术": "19900000001",
        "开始节点": "19900000001",
        "手机号码": "19900000001",
        "结束语发短信": "19900000001",
        "多次未识别发短信": "我19900000001",
        "客户手机号码": "19900000001"
    },
    "call_uuid": "019G2J01JGCV5AO1345H7B5AES074LH5",
    "call_relation_id": "111",
    "reply": "18005699514",
    "business_id": 167,
    "app_id": "1232432",
    "direction": 1
}

~~~
## 响应结果示例
~~~
[{"key":"reportorRelation","type":"integer","value":8}]
注：如果是提供数据给yeta平台，则返回数组里面的对象必须是{"key":"","type":"integer","value":8}格式
其中type的有效类型integer/string。平台推送数据的不做严格要求
~~~
