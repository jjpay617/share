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
| X-Signature   | string | Y       | 签名算法，规则参考下方     |

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
      "data": {},
      "msg": "success"
    }
  
  ```