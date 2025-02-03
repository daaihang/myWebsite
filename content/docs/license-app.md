+++
title = 'License App'
date = 2024-05-31T15:35:03+08:00
draft = true
+++

# 证书系统 开发指北

## 页面管理

- 欢迎页（看板、控制台）
- 证书管理
  - 证书列表颁发/收受证书列表/分享的证书（页面顶部选项卡切换）（只有这页）
  - 添加证书
  - 证书编辑
- 证书模板管理
  - 模板列表（只有这页）
  - 添加模板
  - 模板编辑
- 申请阅办
  - 审核证书
  - 流程管理
- 人员管理
  - 用户列表
  - 组织管理
  - 角色管理
- 个人中心
  - 通知
  - 分享的证书
- 设置
  - 日志


## 角色管理

目前角色暂时写死下面的角色，在`/src/access.ts`进行修改。

每个角色都包含其下级角色的所有权限，同时所有角色都具有用户权限。

1. **系统管理员** - Admin：
   - 负责整个系统的管理和维护。
   - 创建和管理组织。
   - 分配组织管理员权限。
   - 配置系统设置和安全策略。
   - 处理系统级别的问题和故障。

2. **组织管理员** - OrgAdmin：
   - 负责管理特定组织内的用户和操作。
   - 添加、删除和管理组织内的用户。
   - 分配管理员权限给其他用户。
   - 管理组织内用户的角色和权限。
   - 颁发证书给组织内用户。
   - 查看组织内用户的证书。

3. **证书管理员** - LicenseAdmin：
   - 负责证书的管理和颁发。
   - 颁发证书给普通用户。
   - 查看普通用户的证书。
   - 管理普通用户的角色和权限（例如，普通用户是否有权颁发证书）。

4. **普通用户** - User：
   - 接收并查看自己获得的证书。
   - 查看分享给自己的证书。
   - 验证证书的真实性。

## API

### 用户操作（/auth）

1. **POST /auth/register** - 用户注册
2. **POST /auth/login** - 用户登录
3. **POST /auth/logout** - 用户注销
4. **POST /auth/password/reset** - 重置密码请求
5. **POST /auth/password/reset/confirm** - 重置密码
6. **POST /auth/two-factor/enable** - 启用两因素认证
7. **POST /auth/two-factor/disable** - 禁用两因素认证
8. **POST /auth/two-factor/verify** - 验证两因素认证
9. **POST /auth/refresh-token** - 刷新认证令牌

### 用户查询和管理（/user）

1. **GET /user/me** - 获取当前用户信息
2. **PUT /user/me** - 更新当前用户信息
3. **GET /user/{user_id}/logs** - 获取用户日志
4. **GET /user/{user_id}** - 获取指定用户信息
5. **GET /user** - 批量获取用户信息
6. **DELETE /user/{user_id}** - 删除用户
7. **PUT /user/{user_id}** - 更新指定用户信息
8. **PUT /user/{user_id}/deactivate** - 禁用用户账号
9. **PUT /user/{user_id}/activate** - 启用用户账号
10. **POST /user/{user_id}/roles** - 分配角色给用户
11. **DELETE /user/{user_id}/roles** - 从用户移除角色
12. **GET /user/{user_id}/permissions** - 获取用户的所有权限
13. **POST /user/batch** - 批量创建用户
14. **PUT /user/batch** - 批量更新用户
15. **DELETE /user/batch** - 批量删除用户

### 角色操作（/auth/roles）

1. **POST /auth/roles** - 创建角色
2. **PUT /auth/roles/{role_id}** - 更新角色
3. **DELETE /auth/roles/{role_id}** - 删除角色
4. **POST /auth/roles/{role_id}/permissions** - 为角色分配权限
5. **DELETE /auth/roles/{role_id}/permissions** - 移除角色的权限

### 角色查询（/roles）

1. **GET /roles** - 获取所有角色
2. **GET /roles/{role_id}** - 获取指定角色信息
3. **GET /roles/{role_id}/permissions** - 获取指定角色的所有权限

### 权限管理（/auth/permissions）

1. **GET /auth/permissions** - 获取所有权限
2. **POST /auth/permissions** - 创建权限
3. **PUT /auth/permissions/{permission_id}** - 更新权限
4. **DELETE /auth/permissions/{permission_id}** - 删除权限
5. **GET /auth/permissions/{permission_id}** - 获取指定权限信息

### 组织管理（/org）

1. **POST /org** - 创建组织
2. **PUT /org/{org_id}** - 更新组织信息
3. **DELETE /org/{org_id}** - 删除组织
4. **GET /org/{org_id}** - 获取组织信息
5. **GET /org** - 获取所有组织
6. **GET /org/{org_id}/users** - 获取组织下的所有用户
7. **POST /org/{org_id}/users** - 批量添加用户到组织
8. **DELETE /org/{org_id}/users** - 批量从组织中移除用户

### 用户分组管理（/org/{org_id}/groups）

1. **POST /org/{org_id}/groups** - 创建分组
2. **PUT /org/{org_id}/groups/{group_id}** - 更新分组信息
3. **DELETE /org/{org_id}/groups/{group_id}** - 删除分组
4. **GET /org/{org_id}/groups** - 获取所有分组
5. **GET /org/{org_id}/groups/{group_id}** - 获取分组信息
6. **GET /org/{org_id}/groups/{group_id}/users** - 获取分组下的所有用户
7. **POST /org/{org_id}/groups/{group_id}/users** - 批量添加用户到分组
8. **DELETE /org/{org_id}/groups/{group_id}/users** - 批量从分组中移除用户

### 审计日志（/logs）

1. **GET /logs** - 获取所有审计日志
2. **GET /logs/user/{user_id}** - 获取指定用户的审计日志
3. **GET /logs/org/{org_id}** - 获取指定组织的审计日志
4. **GET /logs/role/{role_id}** - 获取指定角色的审计日志

### 安全管理（/security）

1. **POST /security/whitelist** - 添加到IP白名单
2. **DELETE /security/whitelist/{ip}** - 从IP白名单中移除
3. **GET /security/whitelist** - 获取IP白名单
4. **POST /security/blacklist** - 添加到IP黑名单
5. **DELETE /security/blacklist/{ip}** - 从IP黑名单中移除
6. **GET /security/blacklist** - 获取IP黑名单
7. **POST /security/password-policy** - 设置密码策略
8. **GET /security/password-policy** - 获取密码策略
9. **POST /security/account-lock** - 锁定用户账号
10. **POST /security/account-unlock** - 解锁用户账号

### 其他可能的API

1. **GET /metrics** - 获取系统指标
2. **GET /health** - 检查系统健康状态

这些API设计考虑了用户和角色的管理、组织和分组的管理、权限管理、两因素认证、审计日志、IP白名单和黑名单管理、密码策略和账户锁定等功能，确保系统的功能全面且安全。

证书相关

涵盖证书模板管理、证书内容字段管理、证书生成、验证、销毁、导出、分享、通知、历史记录、统计、安全管理和审计等功能。

### 证书模板管理（/certificates/templates）

1. **POST /certificates/templates** - 创建证书模板
2. **PUT /certificates/templates/{template_id}** - 更新证书模板
3. **DELETE /certificates/templates/{template_id}** - 删除证书模板
4. **GET /certificates/templates** - 获取所有证书模板
5. **GET /certificates/templates/{template_id}** - 获取指定证书模板
6. **POST /certificates/templates/{template_id}/clone** - 复制证书模板
7. **POST /certificates/templates/{template_id}/publish** - 发布证书模板
8. **POST /certificates/templates/{template_id}/archive** - 归档证书模板

### 证书内容字段管理（/certificates/templates/{template_id}/fields）

1. **POST /certificates/templates/{template_id}/fields** - 添加证书内容字段
2. **PUT /certificates/templates/{template_id}/fields/{field_id}** - 更新证书内容字段
3. **DELETE /certificates/templates/{template_id}/fields/{field_id}** - 删除证书内容字段
4. **GET /certificates/templates/{template_id}/fields** - 获取证书模板的所有内容字段
5. **GET /certificates/templates/{template_id}/fields/{field_id}** - 获取指定证书内容字段

### 证书管理（/certificates）

1. **POST /certificates** - 颁发证书
2. **POST /certificates/batch** - 批量颁发证书
3. **GET /certificates** - 获取所有证书
4. **GET /certificates/{certificate_id}** - 获取指定证书
5. **DELETE /certificates/{certificate_id}** - 销毁证书
6. **POST /certificates/verify** - 验证证书
7. **POST /certificates/verify/batch** - 批量验证证书
8. **POST /certificates/{certificate_id}/revoke** - 撤销证书
9. **POST /certificates/{certificate_id}/renew** - 更新证书信息并延长有效期
10. **POST /certificates/expire** - 设置证书过期时间
11. **GET /certificates/expired** - 获取已过期证书
12. **GET /certificates/{certificate_id}/status** - 获取证书的状态
13. **POST /certificates/search** - 搜索证书
14. **POST /certificates/{certificate_id}/clone** - 复制证书

### 证书导出和格式转换（/certificates/export）

1. **POST /certificates/{certificate_id}/pdf** - 导出证书为PDF
2. **POST /certificates/export/pdf** - 批量导出证书为PDF
3. **POST /certificates/{certificate_id}/csv** - 导出证书为CSV
4. **POST /certificates/export/csv** - 批量导出证书为CSV

### 证书分享和通知（/certificates/notify）

1. **POST /certificates/{certificate_id}/email** - 将证书通过电子邮件发送
2. **POST /certificates/{certificate_id}/share** - 分享证书给其他用户
3. **POST /certificates/{certificate_id}/notify** - 提醒证书持有者证书即将到期

### 证书历史和统计（/certificates/history, /certificates/stats）

1. **GET /certificates/{certificate_id}/history** - 查看证书的历史记录
2. **GET /certificates/stats** - 获取证书统计信息

### 证书二维码和其他操作（/certificates/qr_code, /certificates/expire, /certificates/renew）

1. **GET /certificates/{certificate_id}/qr_code** - 获取证书的二维码
2. **POST /certificates/expire** - 设置证书过期时间
3. **GET /certificates/expired** - 获取已过期证书
4. **POST /certificates/{certificate_id}/renew** - 更新证书信息并延长有效期

### 证书安全和审计（/certificates/security, /certificates/logs）

1. **GET /certificates/logs** - 获取所有证书的审计日志
2. **GET /certificates/{certificate_id}/logs** - 获取指定证书的审计日志

### 证书权限和授权（/certificates/permissions）

1. **POST /certificates/{certificate_id}/permissions** - 添加证书权限
2. **DELETE /certificates/{certificate_id}/permissions/{permission_id}** - 删除证书权限
3. **GET /certificates/{certificate_id}/permissions** - 获取证书的所有权限

### 证书标签管理（/certificates/tags）

1. **POST /certificates/{certificate_id}/tags** - 添加证书标签
2. **DELETE /certificates/{certificate_id}/tags/{tag_id}** - 删除证书标签
3. **GET /certificates/{certificate_id}/tags** - 获取证书的所有标签

### 证书分类管理（/certificates/categories）

1. **POST /certificates/categories** - 创建证书分类
2. **PUT /certificates/categories/{category_id}** - 更新证书分类
3. **DELETE /certificates/categories/{category_id}** - 删除证书分类
4. **GET /certificates/categories** - 获取所有证书分类
5. **GET /certificates/categories/{category_id}** - 获取指定证书分类
6. **POST /certificates/categories/{category_id}/assign** - 分配证书到分类

### 证书的API覆盖范围

1. **模板管理**：增删查改、克隆、发布、归档。
2. **内容字段管理**：增删查改。
3. **证书管理**：颁发、批量颁发、查、删、验证、批量验证、撤销、续期、过期、状态查询、搜索、克隆。
4. **导出和格式转换**：导出为PDF、CSV，批量导出。
5. **分享和通知**：通过邮件发送、分享给其他用户、到期提醒。
6. **历史和统计**：查看历史记录、获取统计信息。
7. **二维码**：获取证书的二维码。
8. **安全和审计**：获取审计日志。
9. **权限和授权**：添加、删除、查询证书权限。
10. **标签管理**：添加、删除、查询证书标签。
11. **分类管理**：创建、更新、删除、查询分类，分配证书到分类。

这些API涵盖了证书管理系统的各个方面，确保系统功能全面、灵活且安全，能够满足各种证书管理的需求。



## 设置项

### 个人设置（Settings）

- 账户设置
  - 用户名
  - 密码
  - 邮箱绑定
  - 电话绑定
  - 第三方绑定
    - 微信
    - GitHub
    - ~~Google~~
  - 退出登录
  - 注销账号

- 关于
  - 关于 LicenseSys 和 LicenseApp
  - 使用条款和隐私权
  - 版权宣告
  - 取得联络与支援
  - 版本
  - 开源许可证
  - ~~社区参与~~

- 安全设置
  - 修改密码
  - ~~两因素认证设置~~
  - ~~安全问题设置~~
  - ~~登录设备管理~~
  - 最近活动

- 通知设置
  - 邮件通知
  - 短信通知
  - 应用内通知
  - 通知偏好设置

- 外观设置
  - ~~主题选择~~
  - ~~布局设置~~
  - 显示语言
  - ~~字体大小~~
  - 夜间模式

### 系统配置（Config）（仅Admin权限可见）

- 系统基础设置
  - 系统名称
  - 系统时区
  - 默认语言
  - 系统Logo
  - ~~主页配置~~
  - ~~维护模式~~

- 邮箱配置
  - SMTP服务器地址
  - SMTP服务器端口
  - SMTP用户名
  - SMTP密码
  - 发件人地址
  - 邮箱验证模板
  - 密码重置模板

- 密码策略（对新设置的有效）
  - 最小长度
  - 最大长度
  - 密码中需要包含特殊字符
  - 密码有效期
  - ~~密码历史记录~~
  - 失败尝试次数限制

- 用户管理
  - 最大用户数
  - ~~用户角色管理~~
  - ~~用户组管理~~
  - ~~邀请注册设置~~
  - ~~用户注册审批~~

- 数据库配置
  - 数据库类型
  - 数据库连接字符串
  - 数据备份策略
  - 数据恢复

- 日志和审计
  - 日志级别
  - 日志保存路径
  - 日志保留期限
  - 审计日志启用
  - 审计日志查看

- ~~API 配置~~
  - API 访问密钥管理
  - API 速率限制
  - API 端点启用/禁用

- ~~第三方服务~~
  - OAuth提供者配置
  - 外部API集成
  - 第三方插件管理
  - Webhooks配置

- ~~系统性能~~
  - 缓存配置
  - 性能监控
  - 资源限额

- ~~安全设置~~
  - 防火墙设置
  - IP 白名单/黑名单
  - 数据加密设置
  - 安全扫描配置



## 仪表盘

- 参数显示
  - 本用户本月登录次数
  - 本用户颁发/收受证书数量
  - 当前系统中的用户总数
  - 当前系统中的证书总数
  - 待审核证书
- 快捷按钮
  - 创建证书
  - 批量创建证书
  - 审核证书
- 统计图表
  - 用户登录次数趋势图（近30天）
  - 证书颁发/收受数量趋势图（近30天）
  - 不同角色用户数量统计图
- 最新动态
  - 最新用户注册信息
  - 最新证书颁发/收受信息

## 关于 LicenseApp 和 LicenseSys

本系统（证书颁发与验证系统）分为 LicenseApp 和 LicenseSys 部分。LicenseApp 为系统前端部分，使用 Ant Design 构建。LicenseSys 为系统后端部分，使用 Django 构建。