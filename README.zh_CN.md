# @qubit-ltd/common-app

[![npm version](https://badge.fury.io/js/@qubit-ltd/common-app.svg)](https://badge.fury.io/js/@qubit-ltd/common-app)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![English Documentation](https://img.shields.io/badge/docs-English-blue.svg)](README.md)
[![CircleCI](https://dl.circleci.com/status-badge/img/gh/qubit-ltd/js-common-app/tree/master.svg?style=shield)](https://dl.circleci.com/status-badge/redirect/gh/qubit-ltd/js-common-app/tree/master)
[![Coverage Status](https://coveralls.io/repos/github/qubit-ltd/js-common-app/badge.svg?branch=master)](https://coveralls.io/github/qubit-ltd/js-common-app?branch=master)

一个用于开发Web应用程序的JavaScript工具库，提供身份验证存储、具有自动令牌管理的HTTP客户端、用户状态管理以及Web开发的实用函数。

## 特性

- **AuthStorage**: 用户身份验证数据的安全存储管理
- **HTTP客户端**: 增强的Axios实例，具有自动令牌处理和错误管理
- **BasicUserStore**: 基于Pinia的用户状态管理，包含身份验证工作流
- **实用函数**: Web开发常用任务的辅助函数

## 安装

```bash
npm install @qubit-ltd/common-app
```

或者

```bash
yarn add @qubit-ltd/common-app
```

## 快速开始

```javascript
import { AuthStorage, http, BasicUserStore } from '@qubit-ltd/common-app';

// 初始化身份验证存储
const authStorage = new AuthStorage('my-app');

// 使用HTTP客户端
const response = await http.get('/api/users');

// HTTP 模块内部会自动处理文件名提取
```

## API参考

### AuthStorage

用于在本地存储和Cookie中管理用户身份验证数据的类。它为用户凭据、令牌和个人资料信息提供安全存储。

#### 构造函数

```javascript
const authStorage = new AuthStorage(appCode);
```

- `appCode` (string): 应用程序代码，用作存储键的前缀

#### 主要方法

**用户信息管理:**
```javascript
// 用户ID
authStorage.storeUserId(123);
const userId = authStorage.loadUserId();
authStorage.removeUserId();

// 用户名和密码
authStorage.storeUsername('john_doe');
authStorage.storePassword('password123');
const username = authStorage.loadUsername();
const password = authStorage.loadPassword();

// 用户资料
authStorage.storeName('张三');
authStorage.storeAvatar('avatar.jpg');
authStorage.storeGender('MALE');
authStorage.storeMobile('13812345678');
authStorage.storeNickname('小张');
```

**令牌管理:**
```javascript
const token = {
  value: 'jwt-token-here',
  createTime: '2024-01-01T00:00:00Z',
  maxAge: '3600'
};

authStorage.storeToken(token);
const storedToken = authStorage.loadToken();
authStorage.removeToken();

// 检查令牌是否存在
const hasToken = authStorage.hasTokenValue();
const tokenValue = authStorage.loadTokenValue();
```

**权限和角色管理:**
```javascript
authStorage.storePrivileges(['read', 'write', 'admin']);
authStorage.storeRoles(['user', 'moderator']);
const privileges = authStorage.loadPrivileges();
const roles = authStorage.loadRoles();
```

**组织信息:**
```javascript
const organization = {
  id: 1,
  name: '技术部',
  code: 'TECH_DEPT'
};

authStorage.storeOrganization(organization);
const org = authStorage.loadOrganization();
authStorage.removeOrganization();
```

**完整用户会话:**
```javascript
// 存储完整的登录响应
const loginResponse = {
  user: { id: 123, username: 'john', name: '张三' },
  token: { value: 'jwt-token', maxAge: '3600' },
  privileges: ['read', 'write'],
  roles: ['user']
};

authStorage.storeLoginResponse(loginResponse);
const session = authStorage.loadLoginResponse();
authStorage.removeLoginResponse();
```

### http

增强的Axios实例，具有自动令牌管理、错误处理和文件下载功能。

#### 核心特性

- **64位长整数支持**：使用 `@qubit-ltd/json` 自定义序列化器，完美处理后端长整数ID
- **智能错误处理**：自动处理 `LOGIN_REQUIRED`、`SESSION_EXPIRED`、`INVALID_TOKEN` 等认证错误
- **UI抽象层集成**：与 `@qubit-ltd/common-ui` 深度集成，自动显示loading和错误对话框
- **自动令牌管理**：自动添加App Token和Access Token到请求头
- **文件下载功能**：内置文件下载，支持自动解析文件名和MIME类型
- **防缓存机制**：GET请求自动添加时间戳和随机数参数

#### 长整数处理原理

```javascript
// 发送请求时，BigInt自动序列化
const data = { id: 12345678901234567890n, name: 'John' };
// 序列化为: {"id":12345678901234567890,"name":"John"}

// 接收响应时，长整数自动解析为BigInt
const response = '{"id":12345678901234567890,"name":"John Doe"}';
// 解析为: { id: 12345678901234567890n, name: "John Doe" }
```

#### 自动错误处理

HTTP对象可自动处理以下错误类型：

| 错误代码 | 处理方式 | 说明 |
|---------|---------|------|
| `LOGIN_REQUIRED` | 弹出确认登录对话框 | 用户未登录或需要重新认证 |
| `SESSION_EXPIRED` | 根据参数区分处理 | 应用或用户会话过期 |
| `INVALID_TOKEN` | 根据参数区分处理 | 应用或用户令牌无效 |
| `APP_AUTHENTICATION_REQUIRED` | 显示错误信息 | 应用认证失败 |
| 网络错误 | 显示通用错误信息 | 网络连接问题 |

#### 配置要求

在使用HTTP客户端之前，请确保设置了这些配置：

```javascript
import { loading, alert, confirm } from '@qubit-ltd/common-ui';
import config from '@qubit-ltd/config';

// 设置UI框架实现
loading.setImpl(yourLoadingImpl);
alert.setImpl(yourAlertImpl);
confirm.setImpl(yourConfirmImpl);

// 配置HTTP设置
config.set('api_base_url', 'https://api.example.com');
config.set('app_token_value', 'your-app-token');
config.set('http_timeout', 30000);

// 设置令牌管理函数
http.getAccessToken = () => authStorage.loadToken();
http.resetAccessToken = () => authStorage.removeToken();
http.getRouter = () => yourVueRouter;
```

#### 基本用法

```javascript
// GET请求
const users = await http.get('/api/users');

// POST请求带数据
const newUser = await http.post('/api/users', {
  name: '张三',
  email: 'zhangsan@example.com'
});

// PUT请求
const updatedUser = await http.put('/api/users/123', userData);

// DELETE请求
await http.delete('/api/users/123');
```

#### 高级选项

```javascript
// 跳过自动错误处理
try {
  const response = await http.get('/api/data', {
    skipAutoErrorHandling: true
  });
} catch (error) {
  // 手动处理错误
  console.error('请求失败:', error);
}

// 返回完整响应对象
const fullResponse = await http.get('/api/data', {
  returnResponse: true
});
console.log(fullResponse.status, fullResponse.headers);

// 阻止自动清除loading
await http.post('/api/upload', formData, {
  noAutoClearLoading: true
});
```

#### 文件下载

```javascript
// 简单下载
await http.download('/api/files/report.pdf');

// 带参数下载
await http.download('/api/files/export', {
  format: 'xlsx',
  dateRange: '2024-01-01,2024-12-31'
});

// 获取下载信息而不自动下载
const fileInfo = await http.download('/api/files/data.csv', {}, null, false);
console.log(fileInfo.filename, fileInfo.mimeType);
// fileInfo包含: { blob, filename, mimeType }

// 使用自定义文件名下载
await http.download('/api/files/123', {}, 'application/pdf', true, '自定义名称.pdf');
```

### BasicUserStore

用于管理用户身份验证状态和工作流的Pinia store类。

> **📖 详细文档**: [BasicUserStore 用户认证管理详细说明](doc/tutorials/basic-user-store.md) - 查看完整的技术文档和最佳实践

#### 核心特性

- **多种登录方式**: 用户名/密码、手机/验证码、社交网络 Open ID 登录
- **智能状态管理**: 自动管理用户信息、令牌、权限和角色
- **持久化存储**: 使用 AuthStorage 进行本地存储管理
- **令牌验证**: 自动检查令牌有效性和过期状态
- **安全机制**: 提供令牌重置、登录确认等安全功能
- **权限管理**: 集成用户权限和角色管理
- **HTTP 集成**: 与 HTTP 客户端无缝集成，自动令牌注入

#### 依赖注入

BasicUserStore 采用依赖注入模式，需要三个核心依赖：

```javascript
class UserStore extends BasicUserStore {
  constructor() {
    super(
      userAuthenticateApi,  // 用户认证 API 对象
      verifyCodeApi,        // 验证码 API 对象
      appCode              // 应用程序代码
    );
  }
}
```

#### 必需的API接口

| 接口方法 | 说明 | 返回值 |
|---------|------|--------|
| `loginByUsername(username, password)` | 用户名密码登录 | LoginResponse |
| `loginByMobile(mobile, verifyCode)` | 手机验证码登录 | LoginResponse |
| `loginByOpenId(socialNetwork, appId, openId)` | 社交网络登录 | LoginResponse |
| `bindOpenId(socialNetwork, appId, openId)` | 绑定社交账号 | void |
| `logout()` | 用户注销 | void |
| `getLoginInfo()` | 获取登录信息 | LoginResponse |
| `checkToken(userId, tokenValue)` | 检查令牌有效性 | Token \| null |
| `sendBySms(mobile, scene)` | 发送短信验证码 | void |

#### 状态管理

```javascript
// 用户基本信息
userStore.user;          // 当前用户对象 {id, username, nickname, avatar, name, gender, mobile}
userStore.password;      // 用户密码 (仅在记住登录时保存)
userStore.saveLogin;     // 是否保存登录信息
userStore.token;         // 访问令牌 {value, expiredTime}

// 权限和角色
userStore.privileges;    // 用户权限数组
userStore.roles;         // 用户角色数组
userStore.organization;  // 用户所属组织

// 社交登录
userStore.socialNetwork; // 社交网络类型 ('WECHAT' 等)
userStore.appId;         // 社交应用ID
userStore.openId;        // 社交开放ID

// 计算属性
userStore.loggedIn;      // 布尔值：用户是否已登录
```

#### 认证流程示例

```javascript
// 用户名/密码登录
await userStore.loginByUsername('john_doe', 'password123', true);

// 手机号/短信登录
await userStore.sendLoginVerifyCode('13812345678');
await userStore.loginByMobile('13812345678', '123456', false);

// 社交登录
await userStore.loginByOpenId('WECHAT', 'app123', 'openid456');

// 自动恢复登录状态
const token = await userStore.loadToken();
if (token) {
  await userStore.refreshLoginInfo();
}

// 登出
await userStore.logout();
```

#### 权限和角色管理

```javascript
// 检查权限
const hasPermission = (permission) => {
  return userStore.privileges.includes(permission);
};

// 检查角色
const hasRole = (role) => {
  return userStore.roles.includes(role);
};

// 在模板中使用
<q-btn v-if="hasPermission('USER_DELETE')" label="删除用户" />
```

#### 与 HTTP 客户端集成

```javascript
// boot/http.js 中配置
http.getAccessToken = function() {
  const store = useUserStore();
  return store.token;
};

http.resetAccessToken = function() {
  const store = useUserStore();
  store.resetToken();
};
```

## 配置

该库使用`@qubit-ltd/config`进行配置管理。主要配置选项：

```javascript
import config from '@qubit-ltd/config';

// API设置
config.set('api_base_url', 'https://api.example.com');
config.set('http_timeout', 30000);

// 身份验证
config.set('app_token_name', 'X-Auth-App-Token');
config.set('app_token_value', 'your-app-token');
config.set('access_token_name', 'X-Auth-User-Token');

// HTTP头
config.set('http_header_content_type', 'application/json;charset=UTF-8');
config.set('http_header_accept', 'application/json;charset=UTF-8');

// UI设置
config.set('login_page', 'Login');
config.set('social_network', 'WECHAT');
config.set('social_network_app_id', 'your-app-id');
config.set('default_avatar_male', 'male-avatar.png');
config.set('default_avatar_female', 'female-avatar.png');
```

## 错误处理

HTTP客户端为常见场景提供自动错误处理：

- **LOGIN_REQUIRED**: 自动提示用户重新登录
- **TOKEN_EXPIRED**: 处理令牌过期和刷新
- **UNAUTHORIZED**: 重定向到登录页面
- **网络错误**: 显示适当的错误消息

您可以通过在请求选项中设置`skipAutoErrorHandling: true`来禁用自动错误处理。

> 📖 **详细文档**: 查看 [HTTP 对象详细特性和使用方法说明](doc/http-features.md) 了解更多技术细节，包括长整数处理原理、UI抽象层集成、错误处理机制等。

## 完整使用示例

### 1. 基本设置

```javascript
import { createApp } from 'vue';
import { createPinia } from 'pinia';
import { loading, alert, confirm } from '@qubit-ltd/common-ui';
import config from '@qubit-ltd/config';
import { AuthStorage, http, BasicUserStore } from '@qubit-ltd/common-app';

// 配置应用
config.set('api_base_url', 'https://api.myapp.com');
config.set('app_token_value', 'my-app-token-123');
config.set('social_network', 'WECHAT');

// 设置UI实现
loading.setImpl(myLoadingComponent);
alert.setImpl(myAlertComponent);
confirm.setImpl(myConfirmComponent);

// 创建认证存储
const authStorage = new AuthStorage('my-app');

// 配置HTTP客户端
http.getAccessToken = () => authStorage.loadToken();
http.resetAccessToken = () => authStorage.removeToken();
http.getRouter = () => router;

// 创建用户store
const userStore = new BasicUserStore(userApi, verifyCodeApi, 'my-app');
```

### 2. 用户登录流程

```javascript
// 用户名密码登录
async function loginWithPassword(username, password, remember) {
  try {
    await userStore.loginByUsername(username, password, remember);
    console.log('登录成功', userStore.user);
    // 跳转到主页
    router.push('/dashboard');
  } catch (error) {
    console.error('登录失败', error);
  }
}

// 手机号验证码登录
async function loginWithMobile(mobile, code, remember) {
  try {
    await userStore.loginByMobile(mobile, code, remember);
    console.log('登录成功', userStore.user);
  } catch (error) {
    console.error('登录失败', error);
  }
}

// 发送验证码
async function sendVerifyCode(mobile) {
  try {
    await userStore.sendLoginVerifyCode(mobile);
    console.log('验证码已发送');
  } catch (error) {
    console.error('发送验证码失败', error);
  }
}
```

### 3. 文件上传下载

```javascript
// 文件上传
async function uploadFile(file) {
  const formData = new FormData();
  formData.append('file', file);

  try {
    const result = await http.post('/api/upload', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    });
    console.log('上传成功', result);
  } catch (error) {
    console.error('上传失败', error);
  }
}

// 文件下载
async function downloadReport(reportId) {
  try {
    await http.download(`/api/reports/${reportId}/download`);
  } catch (error) {
    console.error('下载失败', error);
  }
}

// 批量下载
async function downloadBatch(ids) {
  try {
    const fileInfo = await http.download('/api/batch-download',
      { ids: ids.join(',') },
      'application/zip',
      false  // 不自动下载
    );

    // 自定义下载逻辑
    const url = URL.createObjectURL(fileInfo.blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = fileInfo.filename || '批量下载.zip';
    a.click();
    URL.revokeObjectURL(url);
  } catch (error) {
    console.error('批量下载失败', error);
  }
}
```

### 4. 权限管理

```javascript
// 检查用户权限
function hasPermission(permission) {
  return userStore.privileges.includes(permission);
}

// 检查用户角色
function hasRole(role) {
  return userStore.roles.includes(role);
}

// 在组件中使用
const canEdit = computed(() => hasPermission('edit'));
const isAdmin = computed(() => hasRole('admin'));
```

## TypeScript支持

该库使用JavaScript编写，但提供TypeScript友好的API。包含类型定义以获得更好的开发体验。

## 依赖项

该库需要以下对等依赖项：

- `@qubit-ltd/common-ui`: 用于loading、alert、confirm的UI组件
- `@qubit-ltd/config`: 配置管理
- `@qubit-ltd/storage`: 存储工具
- `@qubit-ltd/logging`: 日志框架
- `axios`: HTTP客户端
- `pinia`: 状态管理（用于BasicUserStore）

## 许可证

Apache License 2.0

## 贡献

请阅读我们的贡献指南并向仓库提交拉取请求。

## 支持

如有问题和疑问，请使用GitHub issues页面。
