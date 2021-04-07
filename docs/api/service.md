---
id: service
title: 服务端文档
---

## 概述

OpenAPI 采用统一的 Mac Token 头部签算来传递用户身份。

接入客户端  SDK  后，经过用户的授权流程，会获得这个用户在当前应用中的 Mac Token。Mac Token 长期有效，只有在用户更新自己账号相关安全信息、注销对当前应用的授权时才会失效。开发者应当将 Mac Token 妥善保管于自己的服务器上，作为后续与 TapTap 服务端通讯的标示。（Mac Token 算法细节见文档中的 `其他-MacToken` 部分）

以下接口，凡标示类似 `GET` `MAC Token` 的，表示这是一个 `GET` 请求，头部要用 `MAC Token` 签算。

## 流程
1. 移动端用 SDK 的 TapTap 登录，可以通过 `GetCurrentAccessToken` 获取 AccessToken，里面包含
```java
  public String kid;
  public String access_token;
  public String token_type;
  public String mac_key;
  public String mac_algorithm;
  private String json = null;
```

2. 再把 access_token 和 mac_key 发到游戏业务服务器，服务端签算 mac token。
3. 请求 https://openapi.taptap.com/account/profile/v1 ， header 携带 mac token

> 注意：当前实际返回的 kid 和 access_token 值相等，建议使用 access_token

## API

### 获取当前账户详细信息

`GET` `MAC Token` <https://openapi.taptap.com/account/profile/v1>

#### Request

| 字段      | 类型   | 说明   |
| --------- | ------ | ------ |
| client_id | string | 该应用的 Client ID，应与约定相同 |

#### Response

字段             | 类型           | 说明
--------------- | ------------- | ------------
name            | string        | 用户名
avatar          | string        | 用户头像图片
gender         | string       | "female"、"male" 或者空字符串
openid          | string        | 授权用户唯一标识(每个 client 不同)
unionid         | string        | 授权用户唯一标识(每个 union 不同)

#### 请求示例
替换其中的 `MAC id` 和 `client id` 为自己签算的 mac token 和控制台的 Client ID
```
curl -s -H 'Authorization:MAC id="1/hhykMJFMExXBLJrbW823QN3j-O2-MRLBm13XaHNscXgRvLEGQiE2mjXvFIWN_fapPk5dfAcq59kkRD1BUrsocJ1uVWpq5OzGBZ9rwae9-nZ50nzpDLRooFTNT8iTPHmRSH3v0nTk1m4b2-NhXqpGya8t96DQF9zkhf68IkbwIgmDy4GoGVrgVcjFh0xHwLG_4rlwtVR5BZ8-Twyx1PhPjDc8trycgN2i6e-2ivfP6zxrnQr5kW03yQ0QMMgS01Inx4DRcgXMSYPCeNqIxwA6j7WlyrNgU0X0qwnnWBugKOzbJPxA-rgKDu8zVmaly6Xl654V21z2GhWwIfLnil0R6A",ts="1615196300",nonce="abcdef",mac="RgNtmn57fFQB5Ztw7a2KuQyiWkg="' "https://openapi.taptap.com/account/profile/v1?client_id=<clien id>"
```

### 主动撤销 Token

`POST` `MAC Token` <https://www.taptap.com/oauth2/v1/revoke>

只有在当前设备上的主动登出行为，才需要调用撤销接口，如收到 access_denied 401 说明用户在其他渠道撤销了授权，只需要清理本地登录数据

### 授权过期
#### 判断授权过期
服务端请求 OpenAPI 收到 access_denied 错误时，表示 access token 已经失效
SDK 需要在 onError() 的时候手动判断下错误类型，并且提取出错误 message 识别是否为 token 失效

#### 处理授权过期
游戏客户端重新向用户申请授权（即让用户退出登录状态，重新使用 TapTap 登录游戏）

## 其他

### MAC Token 算法

MAC Token 包含以下字段：

| 字段          | 类型   | 说明                            |
| ------------- | ------ | ------------------------------- |
| kid           | string | mac_key id, The key identifier. |
| access_token  | string | 该字段暂无作用                    |
| token_type    | string | Token 类型，如 mac               |
| mac_key       | string | mac 密钥                         |
| mac_algorithm | string | mac计算的算法名称 hmac-sha-1      |

使用 Mac Token 签算一个接口：

#### 脚本
可用此脚本验证直接替换参数，用来验证自己服务端签算的 mac token 是否正确  
CLIENT_ID 替换为控制台获取的 `Client ID`，ACCESS_TOKEN 和 MAC_KEY 为客户端登录成功后的 `access_token`、`mac_key`
```
#!/usr/bin/env bash

# 客户端 ID
CLIENT_ID="请替换为控制台的 Client ID"
# SDK 获取的 access_token
ACCESS_TOKEN="1/hhykMJFMExXBLJrbW823QN3j-O2-MRLBm13XaHNscXgRvLEGQiE2mjXvFIWN_fapPk5dfAcq59kkRD1BUrsocJ1uVWpq5OzGBZ9rwae9-nZ50nzpDLRooFTNT8iTPHmRSH3v0nTk1m4b2-NhXqpGya8t96DQF9zkhf68IkbwIgmDy4GoGVrgVcjFh0xHwLG_4rlwtVR5BZ8-Twyx1PhPjDc8trycgN2i6e-2ivfP6zxrnQr5kW03yQ0QMMgS01Inx4DRcgXMSYPCeNqIxwA6j7WlyrNgU0X0qwnnWBugKOzbJPxA-rgKDu8zVmaly6Xl654V21z2GhWwIfLnil0R6A"
# SDK 获取的 mac_key
MAC_KEY="URpUGfaVHjpQUSeJ2R7pxfVvmY4lEFtgakccSlUQ"

# 随机数，正式上线请替换
NONCE="abcdef"
# 当前时间戳
TS=$(date +%s)

# 请求方法
METHOD="GET"
# 请求地址 (带 query string)
REQUEST_URI="/account/profile/v1?client_id=${CLIENT_ID}"
# 请求域名
REQUEST_HOST="openapi.taptap.com"

MAC=$(printf "%s\n%s\n%s\n%s\n%s\n443\n\n" "${TS}" "${NONCE}" "${METHOD}" "${REQUEST_URI}" "${REQUEST_HOST}" | openssl dgst -binary -sha1 -hmac ${MAC_KEY} | base64)

AUTHORIZATION=$(printf 'MAC id="%s",ts="%s",nonce="%s",mac="%s"' "${ACCESS_TOKEN}" "${TS}" "${NONCE}" "${MAC}")

curl -s -H"Authorization:${AUTHORIZATION}" "https://openapi.taptap.com/account/profile/v1?client_id=${CLIENT_ID}"
```

#### nodejs 代码示例

```javascript
const urllib = require('urllib');
const format = require('string-format');
/**
TapSDK 登录后信息获取
**/
var kid = "";//替换为客户端获取的 kid 或者 access_token
var mac_key = "";//替换为客户端获取的 mac_key
var nonce = "adssd";//自行生成 5 位字符串
var client_id = "";


var ts = Math.ceil(Date.now() / 1000);
var ext = "";//附加信息可以不填
var signArray = [ts, nonce, 'GET', '/account/profile/v1?client_id=' + client_id, 'openapi.taptap.com', 443, ext];

var mac = hmacSha1(signArray.join("\n")+"\n", mac_key);
var auth = format('MAC id={id},ts={ts},nonce={nonce},mac={mac}', {
  id: '\"'+kid+'\"',
  ts: '\"'+ts+'\"',
  nonce: '\"'+nonce+'\"',
  mac: '\"'+mac+'\"'
});

var headers = {
  Authorization: auth
}

var reqData = {
  method: "GET",
  headers: headers
}

urllib.request("https://openapi.taptap.com/account/profile/v1?client_id=" + client_id, reqData,
  (err, data, response) => {
    if(!err){
      console.log("返回数据：" + data.toString());
    }
  });

function hmacSha1(encodedFlags, secretKey) {
    var hmac = crypto.createHmac('sha1', secretKey);
    hmac.update(encodedFlags);
    return hmac.digest('base64');
};

```



### 通用接口错误信息

**统一格式**

| 字段              | 类型   | 说明                                                 |
| ----------------- | ------ | ---------------------------------------------------- |
| code              | int    | 预留字段，用于以后追踪问题                           |
| error             | string | 错误码，代码逻辑判断时使用                           |
| error_description | string | 错误描述信息，开发的时候用来帮助理解和解决发生的错误 |


**错误响应**

| 错误码                    | HTTP 状态码 | 详细描述                                                     |
| ------------------------- | ----------- | ------------------------------------------------------------ |
| invalid_request           | 400         | 请求缺少某个必需参数，包含一个不支持的参数或参数值，或者格式不正确 |
| invalid_time              | 400         | MAC Token 算法中，ts 时间不合法，**应请求服务器时间重新构造** |
| invalid_client            | 401         | client_id、client_secret 参数无效                             |
| access_denied             | 401         | 授权服务器拒绝请求 **这个状态出现在拿着 token 请求用户资源时，如出现，客户端应退出本地的用户登录信息，引导用户重新登录** |
| forbidden       | 403         | 用户没有对当前动作的权限，**引导重新身份验证并不能提供任何帮助，而且这个请求也不应该被重复提交** |
| not_found       | 404         | 请求失败，请求所希望得到的资源未被在服务器上发现。**在参数相同的情况下，不应该重复请求** |
| server_error              | 500         | 服务器出现异常情况 **可稍等后重新尝试请求，但需有尝试上限，建议最多 3 次，如一直失败，则中断并告知用户** |