# 服务器与服务器的即时通信协议 V0.1

服务器与服务器之间通信协议使用http协议传输，服务器端与客户端使用tcp协议，内容放在http的body中，以json形式予以存放，以下是一个服务器间消息实例：  
![sample](https://user-images.githubusercontent.com/5525436/42436872-f00af064-838d-11e8-8445-4f197b88508b.png)

```json
{
"type":"message",
"from":"inquiry@polyglot.net.cn",
"to":"customer@w2v4.com",
"body":"你好吗？",
"version":0.1
}
```
type类型有：  
### search - 搜索类型
定义搜索用户请求及响应  
```json
{
"type":"search",
"action":"request",
"user":"zxj@polyglot.net.cn",
"version":0.1
}
```
```json
{
"type":"search",
"action":"response",
"user":{"id":"zxj@polyglot.net.cn","nickname":"哲学家"},
"version":0.1
}
```
查询无结果
```json
{
"type":"search",
"action":"response",
"user":null,
"version":0.1
}
```
### message - 消息类型
发送消息
```json
{
"type":"message",
"from":"inquiry@polyglot.net.cn",
"to":"customer@w2v4.com",
"body":"你好吗？",
"version":0.1
}
```

### friend - 好友类型  
#### 添加好友  
```json
{
"type":"friend",
"action":"request",
"from":"zxj@polyglot.net.cn",
"to":"customer@w2v4.com",
"message":"请添加我为你的好友，我是哲学家",
"version":0.1
}
```
#### 删除好友  
```json
{
"type":"friend",
"action":"delete",
"from":"zxj@polyglot.net.cn",
"to":"customer@w2v4.com",
"version":0.1
}
```
#### 答复好友请求
> 以下的 json 结果表示 `to` 用户 接受了 `from` 用户的好友请求。
```json
{
"type":"friend",
"action":"response",
"from":"zxj@polyglot.net.cn",
"to":"customer@w2v4.com",
"accept":true,
"version":0.1
}
```
#### 获取好友列表
请求以响应  
```json
{
"type":"friend",
"action":"list",
"from":"zxj@polyglot.net.cn",
"version":0.1
}
```
结果  
```json
{
"type": "friend",
"action": "list",
"from": "zxj@polyglot.net.cn",
"version": 0.1,
"results": [
  {
    "group": "我的好友",
    "lists": [
    {
      "id": "test123@polyglot.net.cn",
      "nickname": "test123@polyglot.net.cn"
    },
    {
      "id": "test233@polyglot.net.cn",
      "nickname": "test233@polyglot.net.cn"
    }
    ]
  }]
}
```
json 结果 `d.ts` 描述。
```typescript
class RootObject {
  type: string;
  action: string;
  from: string;
  version: number;
  results: Result[];
}
class Result {
  group: string;
  lists: User[];
}
class User {
  id: string;
  nickname: string;
}
```

### user - 用户类型
> - 传输时 `crypto` 为加密后的密码内容，使用 `MD5` 进行加密。如有需要请酌情增加加密选项。  
> - 示例中的 `crypto` 的值为 `zxj5470` 的 MD5 加密后的内容

#### 用户登录
```json
{
"type":"user",
"action":"login",
"user":"zxj5470",
"crypto":"431fe828b9b8e8094235dee515562247",
"version":0.1
}
```
登录返回结果  
```json
{
"type":"user",
"action":"login",
"user":{
  "id":"zxj5470"
  },
"version":0.1,
"login":true
}
```

#### 用户注册
```json
{
"type":"user",
"action":"registry",
"user":"zxj5470",
"crypto":"431fe828b9b8e8094235dee515562247",
"version":0.1
}
```
返回结果  
```json
{
"type":"user",
"action":"registry",
"user":"zxj5470",
"version":0.1,
"info":"注册成功",
"registry":true
}
```

#### 错误样例：   
- 用户名长度小于 4。
- 密码格式不正确（验证 MD5 长度为 32）
相关说明  
- 有对应通信客户端时建议直接在客户端部分做好验证。但若提供 Web Service API 则提示如下信息。  
```json
{
  "type": "user",
  "action": "login",
  "user": " ",
  "version": 0.1,
  "info": "用户名格式错误",
  "registry": false
}
```