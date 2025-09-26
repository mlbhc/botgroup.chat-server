# BotGroup Chat API 接口文档

## 概述

BotGroup Chat API Server 是一个基于 Gin 框架的 Go 语言后端服务，提供聊天机器人群组管理、用户认证、文件上传等功能。

**服务器信息：**
- 端口：8080
- 版本：1.0.0
- 框架：Gin

## 基础信息

### 服务器状态检查

#### 1. 根路径
- **接口地址：** `GET /`
- **说明：** 获取API服务器基本信息
- **认证：** 无需认证

**响应示例：**
```json
{
  "message": "BotGroup Chat API Server",
  "version": "1.0.0",
  "status": "running"
}
```

#### 2. 健康检查（简单）
- **接口地址：** `GET /health`
- **说明：** 简单的健康检查端点（用于Docker健康检查）
- **认证：** 无需认证

**响应示例：**
```json
{
  "status": "ok",
  "message": "服务正常运行"
}
```

#### 3. 健康检查（详细）
- **接口地址：** `GET /health/detailed`
- **说明：** 详细健康检查，包含数据库连接状态
- **认证：** 无需认证

**成功响应示例：**
```json
{
  "status": "ok",
  "message": "服务正常运行",
  "database": "connected"
}
```

**失败响应示例：**
```json
{
  "status": "error",
  "message": "数据库连接失败",
  "error": "具体错误信息"
}
```

## 认证相关接口

### 1. 用户登录
- **接口地址：** `POST /api/login`
- **说明：** 手机号验证码登录
- **认证：** 无需认证

**请求参数：**
```json
{
  "phone": "13800138000",
  "code": "123456"
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "登录成功",
  "data": {
    "token": "jwt_token_string",
    "user": {
      "id": 1,
      "phone": "13800138000",
      "nickname": "用户昵称",
      "avatar_url": "头像URL",
      "status": 1,
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z",
      "last_login_at": "2024-01-01T00:00:00Z"
    }
  }
}
```

### 2. 获取验证码
- **接口地址：** `GET /api/captcha`
- **说明：** 获取图形验证码
- **认证：** 无需认证

**响应：** 返回验证码图片数据

### 3. 验证码校验并发送短信
- **接口地址：** `POST /api/captcha/check`
- **说明：** 验证图形验证码并发送短信验证码
- **认证：** 无需认证

**请求参数（Form Data）：**
```
dots: "点击坐标数据"
key: "验证码key"
extraData: "{\"phone\":\"13800138000\"}"
```

**响应示例：**
```json
{
  "code": 0,
  "message": "验证码验证通过，短信已发送",
  "success": true
}
```

## 微信登录相关接口

### 1. 生成微信二维码
- **接口地址：** `POST /api/auth/wechat/qr-code`
- **说明：** 生成微信扫码登录二维码
- **认证：** 无需认证
- **限流：** 微信二维码限流

**请求参数（可选）：**
```json
{
  "redirect_uri": "登录成功后跳转地址"
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "二维码生成成功",
  "data": {
    "qr_url": "二维码图片URL",
    "session_id": "会话ID",
    "qr_scene": "二维码场景值",
    "expires_in": 600
  }
}
```

### 2. 微信回调处理
- **接口地址：** `GET|POST /api/auth/wechat/callback`
- **说明：** 微信服务器回调处理
- **认证：** 微信签名验证
- **限流：** 微信回调限流

**GET请求（服务器验证）：**
- 查询参数：signature, timestamp, nonce, echostr
- 响应：返回echostr验证成功

**POST请求（事件处理）：**
- 请求体：微信XML格式消息
- 响应：XML格式回复消息

### 3. 查询微信登录状态
- **接口地址：** `GET /api/auth/wechat/status/:session_id`
- **说明：** 查询微信扫码登录状态
- **认证：** 无需认证
- **限流：** 微信状态查询限流

**响应示例（等待扫码）：**
```json
{
  "success": true,
  "status": "pending",
  "message": "等待扫码"
}
```

**响应示例（登录成功）：**
```json
{
  "success": true,
  "status": "success",
  "message": "登录成功",
  "data": {
    "user": {
      "id": 1,
      "nickname": "用户昵称",
      "avatar": "头像URL"
    },
    "token": "jwt_token",
    "expires_in": 604800
  }
}
```

### 4. 微信登录测试接口（仅开发环境）
- **接口地址：** `GET /api/auth/wechat/test`
- **说明：** 测试微信登录流程
- **认证：** 无需认证（仅开发环境）
- **限流：** 微信二维码限流

**查询参数：**
- session_id: 会话ID
- openid: 微信OpenID

**响应示例：**
```json
{
  "success": true,
  "message": "模拟登录成功",
  "data": {
    "user_id": 1,
    "session_id": "session_id"
  }
}
```

### 5. 微信Token调试接口（仅开发环境）
- **接口地址：** `GET /api/auth/wechat/debug/token`
- **说明：** 调试微信Access Token状态
- **认证：** 无需认证（仅开发环境）

**响应示例：**
```json
{
  "success": true,
  "message": "Token状态查询成功",
  "data": {
    "valid": true,
    "token": "access_token",
    "expires_in": 7200
  }
}
```

## 匿名聊天接口

### 1. 初始化接口
- **接口地址：** `GET /api/init`
- **说明：** 获取应用初始化配置信息
- **认证：** 无需认证
- **限流：** 聊天限流

**响应示例：**
```json
{
  "code": 200,
  "message": "成功",
  "data": {
    "models": ["模型列表"],
    "groups": ["群组列表"],
    "characters": ["角色列表"],
    "user": "用户信息（如果已登录）"
  }
}
```

### 2. 聊天接口
- **接口地址：** `POST /api/chat`
- **说明：** 发送聊天消息，支持流式输出
- **认证：** 无需认证
- **限流：** 聊天限流

**请求参数：**
```json
{
  "message": "用户消息内容",
  "user_id": "用户ID（可选）",
  "history": [
    {
      "user_id": "用户ID",
      "role": "user|assistant",
      "name": "角色名称",
      "content": "消息内容",
      "timestamp": "时间戳"
    }
  ]
}
```

**响应：** Server-Sent Events (SSE) 流式数据
```
data: {"content": "AI回复内容片段"}
data: {"content": "继续的内容"}
data: {"error": "错误信息"}
```

## 需要认证的用户接口

> 以下接口需要在请求头中携带 JWT Token：
> `Authorization: Bearer <token>`

### 用户相关接口

#### 1. 获取用户信息
- **接口地址：** `GET /api/user/info`
- **说明：** 获取当前登录用户信息
- **认证：** 需要JWT Token

**响应示例：**
```json
{
  "success": true,
  "message": "获取用户信息成功",
  "data": {
    "id": 1,
    "phone": "13800138000",
    "nickname": "用户昵称",
    "avatar_url": "头像URL",
    "status": 1,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "last_login_at": "2024-01-01T00:00:00Z"
  }
}
```

#### 2. 更新用户信息
- **接口地址：** `POST /api/user/update`
- **说明：** 更新用户昵称和头像
- **认证：** 需要JWT Token

**请求参数：**
```json
{
  "nickname": "新昵称",
  "avatar_url": "新头像URL"
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "更新用户信息成功",
  "data": {
    "id": 1,
    "phone": "13800138000",
    "nickname": "新昵称",
    "avatar_url": "新头像URL",
    "status": 1,
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "last_login_at": "2024-01-01T00:00:00Z"
  }
}
```

#### 3. 文件上传
- **接口地址：** `POST /api/user/upload`
- **说明：** 获取Cloudflare上传URL
- **认证：** 需要JWT Token

**响应示例：**
```json
{
  "success": true,
  "message": "获取上传URL成功",
  "data": {
    "id": "上传ID",
    "uploadURL": "上传URL",
    "result": "Cloudflare返回的完整结果"
  }
}
```

### AI调度接口

#### 1. AI响应调度
- **接口地址：** `POST /api/scheduler`
- **说明：** 根据消息内容调度合适的AI进行回复
- **认证：** 需要JWT Token

**请求参数：**
```json
{
  "message": "用户消息",
  "history": [
    {
      "user_id": "用户ID",
      "role": "user|assistant",
      "name": "角色名称",
      "content": "消息内容",
      "timestamp": "时间戳"
    }
  ],
  "availableAIs": [
    {
      "id": "AI ID",
      "name": "AI名称",
      "personality": "性格描述",
      "model": "模型名称",
      "avatar": "头像URL",
      "custom_prompt": "自定义提示词"
    }
  ]
}
```

**响应示例：**
```json
{
  "selectedAIs": ["选中的AI ID列表"]
}
```

## 群组管理接口

### 1. 创建群组
- **接口地址：** `POST /api/groups`
- **说明：** 创建新的AI群组
- **认证：** 需要JWT Token

**请求参数：**
```json
{
  "name": "群组名称",
  "description": "群组描述"
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "创建群组成功",
  "data": {
    "id": 1,
    "name": "群组名称",
    "description": "群组描述",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  }
}
```

### 2. 获取群组列表
- **接口地址：** `GET /api/groups`
- **说明：** 获取群组列表（支持分页和搜索）
- **认证：** 需要JWT Token

**查询参数：**
- page: 页码（默认1）
- page_size: 每页数量（默认10，最大100）
- name: 群组名称搜索

**响应示例：**
```json
{
  "success": true,
  "message": "获取群组列表成功",
  "data": [
    {
      "id": 1,
      "name": "群组名称",
      "description": "群组描述",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 1
}
```

### 3. 获取单个群组详情
- **接口地址：** `GET /api/groups/:id`
- **说明：** 获取指定群组的详细信息
- **认证：** 需要JWT Token

**响应示例：**
```json
{
  "success": true,
  "message": "获取群组详情成功",
  "data": {
    "id": 1,
    "name": "群组名称",
    "description": "群组描述",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "characters": [
      {
        "id": 1,
        "name": "角色名称",
        "personality": "角色性格",
        "model": "AI模型",
        "avatar": "头像URL",
        "custom_prompt": "自定义提示词"
      }
    ]
  }
}
```

### 4. 更新群组
- **接口地址：** `PUT /api/groups/:id`
- **说明：** 更新群组信息
- **认证：** 需要JWT Token

**请求参数：**
```json
{
  "name": "新群组名称",
  "description": "新群组描述"
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "更新群组成功",
  "data": {
    "id": 1,
    "name": "新群组名称",
    "description": "新群组描述",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  }
}
```

### 5. 删除群组
- **接口地址：** `DELETE /api/groups/:id`
- **说明：** 删除指定群组（会级联删除相关角色）
- **认证：** 需要JWT Token

**响应示例：**
```json
{
  "success": true,
  "message": "删除群组成功"
}
```

### 6. 获取群组下的角色列表
- **接口地址：** `GET /api/groups/:id/characters`
- **说明：** 获取指定群组下的所有角色
- **认证：** 需要JWT Token

**查询参数：**
- page: 页码（默认1）
- page_size: 每页数量（默认10，最大100）

**响应示例：**
```json
{
  "success": true,
  "message": "获取群组角色列表成功",
  "group": {
    "id": 1,
    "name": "群组名称",
    "description": "群组描述"
  },
  "data": [
    {
      "id": 1,
      "gid": 1,
      "name": "角色名称",
      "personality": "角色性格",
      "model": "AI模型",
      "avatar": "头像URL",
      "custom_prompt": "自定义提示词",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z"
    }
  ],
  "total": 1
}
```

## 角色管理接口

### 1. 创建角色
- **接口地址：** `POST /api/characters/`
- **说明：** 创建新的AI角色
- **认证：** 需要JWT Token

**请求参数：**
```json
{
  "gid": 1,
  "name": "角色名称",
  "personality": "角色性格描述",
  "model": "AI模型名称",
  "avatar": "头像URL",
  "custom_prompt": "自定义提示词"
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "创建角色成功",
  "data": {
    "id": 1,
    "gid": 1,
    "name": "角色名称",
    "personality": "角色性格描述",
    "model": "AI模型名称",
    "avatar": "头像URL",
    "custom_prompt": "自定义提示词",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  }
}
```

### 2. 获取角色列表
- **接口地址：** `GET /api/characters/`
- **说明：** 获取角色列表（支持分页和搜索）
- **认证：** 需要JWT Token

**查询参数：**
- page: 页码（默认1）
- page_size: 每页数量（默认10，最大100）
- gid: 群组ID筛选
- name: 角色名称搜索
- model: AI模型筛选

**响应示例：**
```json
{
  "success": true,
  "message": "获取角色列表成功",
  "data": [
    {
      "id": 1,
      "gid": 1,
      "name": "角色名称",
      "personality": "角色性格",
      "model": "AI模型",
      "avatar": "头像URL",
      "custom_prompt": "自定义提示词",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z",
      "group": {
        "id": 1,
        "name": "群组名称",
        "description": "群组描述"
      }
    }
  ],
  "total": 1
}
```

### 3. 获取单个角色详情
- **接口地址：** `GET /api/characters/:id`
- **说明：** 获取指定角色的详细信息
- **认证：** 需要JWT Token

**响应示例：**
```json
{
  "success": true,
  "message": "获取角色详情成功",
  "data": {
    "id": 1,
    "gid": 1,
    "name": "角色名称",
    "personality": "角色性格",
    "model": "AI模型",
    "avatar": "头像URL",
    "custom_prompt": "自定义提示词",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z",
    "group": {
      "id": 1,
      "name": "群组名称",
      "description": "群组描述"
    }
  }
}
```

### 4. 更新角色
- **接口地址：** `PUT /api/characters/:id`
- **说明：** 更新角色信息
- **认证：** 需要JWT Token

**请求参数：**
```json
{
  "name": "新角色名称",
  "personality": "新角色性格",
  "model": "新AI模型",
  "avatar": "新头像URL",
  "custom_prompt": "新自定义提示词"
}
```

**响应示例：**
```json
{
  "success": true,
  "message": "更新角色成功",
  "data": {
    "id": 1,
    "gid": 1,
    "name": "新角色名称",
    "personality": "新角色性格",
    "model": "新AI模型",
    "avatar": "新头像URL",
    "custom_prompt": "新自定义提示词",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  }
}
```

### 5. 删除角色
- **接口地址：** `DELETE /api/characters/:id`
- **说明：** 删除指定角色
- **认证：** 需要JWT Token

**响应示例：**
```json
{
  "success": true,
  "message": "删除角色成功"
}
```

## WebSocket 接口

### WebSocket 连接
- **接口地址：** `GET /ws/auth/:session_id`
- **说明：** 建立WebSocket连接，用于实时通信
- **认证：** 通过session_id验证

**连接URL示例：**
```
ws://localhost:8080/ws/auth/your_session_id
```

**消息格式：**
```json
{
  "type": "login_result",
  "data": {
    "status": "success|failed|expired",
    "message": "状态描述",
    "user_info": {
      "user_id": 1,
      "nickname": "用户昵称",
      "avatar_url": "头像URL",
      "login_type": "wechat"
    },
    "token": "jwt_token",
    "expires_in": 604800
  }
}
```

## 错误代码说明

### HTTP状态码
- `200 OK`: 请求成功
- `400 Bad Request`: 请求参数错误
- `401 Unauthorized`: 未授权/认证失败
- `403 Forbidden`: 禁止访问
- `404 Not Found`: 资源不存在
- `405 Method Not Allowed`: 请求方法不允许
- `500 Internal Server Error`: 服务器内部错误
- `503 Service Unavailable`: 服务不可用

### 业务错误码
响应中的 `code` 字段：
- `0`: 成功
- `1`: 业务逻辑错误

### 常见错误信息
- "请求参数无效": 请求体JSON格式错误或必填字段缺失
- "用户认证失败": JWT Token无效或过期
- "验证码验证失败": 图形验证码错误
- "手机号格式无效": 手机号格式不正确
- "短信发送失败": 短信服务异常
- "群组不存在": 指定的群组ID不存在
- "角色不存在": 指定的角色ID不存在

## 中间件说明

### 认证中间件
- **作用域：** `/api/` 下的用户相关接口
- **功能：** 验证JWT Token，解析用户信息

### CORS中间件
- **作用域：** 全局
- **功能：** 处理跨域请求

### 限流中间件
- **聊天限流：** 限制聊天接口的访问频率
- **微信相关限流：** 
  - 二维码生成限流
  - 回调处理限流
  - 状态查询限流

### 安全头中间件
- **作用域：** 认证相关接口
- **功能：** 添加安全相关的HTTP头

### 微信签名验证中间件
- **作用域：** 微信POST回调
- **功能：** 验证微信服务器签名

## 注意事项

1. **认证Token：** 需要认证的接口必须在请求头中携带有效的JWT Token
2. **分页参数：** 列表接口支持分页，page_size最大值为100
3. **文件上传：** 使用Cloudflare Images服务，需要配置相关参数
4. **开发环境接口：** 部分调试接口仅在开发环境（端口8080）可用
5. **限流机制：** 部分接口有访问频率限制，请合理控制请求频率
6. **WebSocket连接：** 用于实时推送登录状态变更
7. **流式响应：** 聊天接口使用Server-Sent Events返回流式数据

## 示例代码

### JavaScript 调用示例

```javascript
// 登录
const login = async (phone, code) => {
  const response = await fetch('/api/login', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ phone, code })
  });
  return response.json();
};

// 获取用户信息（需要token）
const getUserInfo = async (token) => {
  const response = await fetch('/api/user/info', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  });
  return response.json();
};

// 流式聊天
const chat = async (message) => {
  const eventSource = new EventSource('/api/chat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ message })
  });
  
  eventSource.onmessage = (event) => {
    const data = JSON.parse(event.data);
    console.log('收到消息:', data);
  };
};

// WebSocket连接
const connectWebSocket = (sessionId) => {
  const ws = new WebSocket(`ws://localhost:8080/ws/auth/${sessionId}`);
  
  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.type === 'login_result') {
      console.log('登录结果:', data.data);
    }
  };
};
```
