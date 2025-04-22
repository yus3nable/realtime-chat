# API 文档

---

## 登录

- **URL**: `/api/login`
- **方法**: `POST`
- **请求头**:
  - `Content-Type: application/json`

### 请求体
```json
{
  "username": "zhang_san@example.com",   // 用户邮箱或用户名
  "password": "my_secure_p@ssw0rd",      // 密码
  "remember_me": true                    // 可选，是否记住登录状态
}
```

### 成功响应 (200 OK)
```json
{
  "status": "success",                   
  "message": "登录成功",                 
  "user": {
    "id": 12345,
    "username": "zhang_san",
    "email": "zhang_san@example.com"
  },
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."  // JWT，用于后续鉴权
}
```

### 失败响应
- **400 Bad Request**
  ```json
  {
    "status": "error",
    "message": "登录尝试次数过多，需要验证码",
    "error_code": "captcha_required",
    "captcha_url": "/api/captcha/generate?id=xyz123"
  }
  ```
- **401 Unauthorized**
  ```json
  {
    "status": "error",
    "message": "用户名或密码不正确",
    "error_code": "invalid_credentials"
  }
  ```

### 注意事项
- 登录成功后，所有受保护接口请在请求头中添加 `Authorization: Bearer {token}`
- token 有效期 24 小时，过期后需重新登录获取新 token
- 连续 5 次登录失败后，返回 400 并要求图形验证码

---

## 群聊消息 (WebSocket)

- **URL**: `wss://example.com/api/chat`
- **鉴权**: 连接时请在握手阶段带上 HTTP 头 `Authorization: Bearer {token}`，或用 `?token=` 查询串（二选一）

### 输入（Client → Server）
```json
{
  "type": "message",          
  "room_id": "group123",      
  "content": "你好，大家好！", 
  "timestamp": 1650372465000  
}
```

### 输出（Server → Clients in room）
```json
{
  "type": "new_message",      
  "message_id": "msg_12345",  
  "room_id": "group123",      
  "sender_id": 10001,          
  "sender_name": "zhang_san", 
  "content": "你好，大家好！",
  "timestamp": 1650372465000  
}
```

### 说明
- 服务端根据 Authorization 中的 token 确定 sender_id/sender_name，客户端无需在输入里带用户信息
- message_id 由服务端生成，保证全局唯一
- timestamp 建议由客户端和服务端同时赋值，避免时差引起乱序
- type 用于区分消息类别，客户端看到 new_message 时要把它渲染到对应房间
