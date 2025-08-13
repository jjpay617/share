# 商户开发者文档


## 约定声明

所有 `{}` 内的值需要根据实际情况进行替换。注意: `{id}`是单个id值，`{ids}`是一组id值中间使用`,`分隔

### 各环境基地址

| 名称  | 地址                          |
|-----|-----------------------------|
| 生产环境（prod） | https://psbajzjk.fyvfjo.com |

### HTTP 状态码

| code | 描述    |
|------|-------|
| 200  | 正常    |
| 401  | 未授权   |
| 403  | 权限不足  |
| 404  | 资源不存在 |


### 响应数据结构

- **Success**: HTTP 200 OK

  ```json5
  
  {
    "code": 200,
    "data": "datas...",
    "msg": "success"
  }
  
  ```

- **Fail**: HTTP 200 OK

  ```json5
  
  {
    "code": 401015,
    "data": "",
    "msg": "参数传递无效"
  }
  
  ```

### 公共参数

**Request Headers**

| 参数            | 类型     | 必填      | 描述                     |
|---------------|--------|---------|------------------------|
| Content-Type  | string | Y       | 固定值: `application/json` |
| X-Merchant-ID | int    | Y       | 商户ID                   |
| X-Timestamp   | int    | Y       | 当前时间戳单位为秒，误差不能超过5分钟 |
| X-Signature   | string | Y       | 签名，算法规则参考下方     |

> **`X-Signature` 签名算法**  
> 采用 **HMAC-SHA256** 算法，使用分配的商户密钥进行签名（商户秘钥及 merchantID 在商户后台获取）
>
> **GET 请求签名构造**
> ```
> 待签名字符串 = merchantID + timestamp + query_params
> 请求示例: https://api.example.com/v1/query?param1=value1&param2=value2
> ```
>
> **POST 请求签名构造**
> ```
> 待签名字符串 = merchantID + timestamp + query_params + request_body
> 请求示例: https://api.example.com/v1/create?param1=value1
> 请求体: {"name": "test","value": "123"}
> ```
>
> **提示**
> - query_params 参数按 key 升序排序，其值进行 URL 编码
> - POST 请求体必须是 JSON 格式（无需格式化）
> - 时间戳误差不能超过 5 分钟
> - 签名结果大小写敏感

## 订单状态码

| code | 描述    |
|------|-------|
| 1    | 待支付    |
| 2    | 已支付   |
| 3    | 已完成  |
| 4    | 已失败 |
| 5    | 已取消 |
| 6    | 申诉中 |
| 7    | 待释放 |

## 发起代付

### POST `/v1/exchange/ad/merchant/sell/up`

| 参数                            | 类型     | 必填 | 描述                     |
|-------------------------------|--------|----|------------------------|
| symbolid | int    | Y  | 代币符号ID，固定值：`104`       |
| finance | object | Y  | 收款信息，`object` 数据结构参考下方 |
| method | string | Y  | 撮合方式，固定值：`api`         |
| out_orderno | string | Y  | 商户订单号                  |
| amount | string | Y  | 金额                     |

> 支付宝

| 参数       | 类型       | 必填 | 描述             |
|----------|----------|----|----------------|
| bank_no     | string      | Y  | 支付宝账号          |
| bank_name  | string      | Y  | 支付方式，固定值：`支付宝` |
| receiver  | string      | Y  | 收款人姓名          |

> 银行卡

| 参数       | 类型       | 必填 | 描述             |
|----------|----------|----|----------------|
| bank_no     | string      | Y  | 银行卡账号          |
| bank_name  | string      | Y  | 支付方式，固定值：`银行卡` |
| receiver  | string      | Y  | 收款人姓名          |

### 响应

- **Success**: HTTP 200 OK

  ```json5
  
    {
      "code": 200,
      "data": {
          "ad": {
              "id": 1000000
          },
          "order": {
              "orderno": "BE7361265191402606592",
              "status": 1,
              "out_orderno": "TEST7361265191402606592"
          }
      },
      "msg": "success"
  }
  
  ```

## 回调通知

- 商户需进入`商户系统后台->安全中心->设置回调地址`配置回调通知地址
- 平台将订单状态回调给商户系统，商户收到回调后，需返回大写`SUCCESS`以表示确认收到回调
- 重试机制：如果平台未收到 SUCCESS 响应标识，会每隔1分钟重新回调，若连续 10 次 回调失败，将不会再尝试自动回调

### POST `商户设置的回调地址`

| 参数                            | 类型     | 必填 | 描述                 |
|-------------------------------|--------|----|--------------------|
| type | string | Y  | 通知类型，值参考下方         |
| business | string | Y  | 业务类型，`3` 交易，`4` 代付 |
| ad_id | int    | Y  | 代付ID               |
| dispute_result_action | int    | N  | 申诉处理结果，值参考下方       |
| orderno | string | Y  | 订单号                |
| out_orderno | string | Y  | 商户订单号              |

> type 通知类型

| type | 描述       |
|------|----------|
| ORDER_CALLBACK_PENDING    | 订单进行中    |
| ORDER_CALLBACK_CANCEL    | 订单取消     |
| ORDER_CALLBACK_COMPLETED    | 订单完成     |
| ORDER_CALLBACK_PAID    | 订单已支付    |
| ORDER_CALLBACK_DISPUTE_RESULT    | 订单申诉处理结果 |
| ORDER_CALLBACK_RELEASING    | 释放代币     |

> dispute_result_action 申诉处理结果

| dispute_result_action         | 描述       |
|-------------------------------|----------|
| 1                             | 驳回申诉    |
| 2                             | 取消订单     |
| 3                             | 完成订单    |

### 商户应答

- **Success**: HTTP 200 OK

  ```text
  
    SUCCESS
  
  ```