---
title: "Supabase + RLS + Service Role Key ——“人话总结”"
date: 2025-07-13
---

### ✅ Supabase + RLS + Service Role Key ——“人话总结”：

Supabase 背后是 PostgreSQL，而 PostgreSQL 提供了非常强大的 **行级安全机制（RLS）**。
Supabase 默认开启了这个功能，目的是：

> **让用户只能看到和操作属于自己的数据，防止越权。**

所以，我们会在数据库表上写 policy，比如：

```sql
user_id = auth.uid()
```

确保用户只能访问自己的行。

---

### 😓 那后台怎么办？

问题是：

* 后台处理往往是“跨用户的”操作（比如定时任务修改所有用户状态、平台发系统通知等）
* 管理员页面操作也不属于某个用户的 session（比如超级管理员审核订单）

这时你就 **无法用前端 session 中的 `auth.uid()` 来通过 RLS 的检查**，哪怕你是管理员也被挡住了。

---

### 🗝️ Service Role Key 是什么？

> **就是一把万能钥匙，告诉数据库：“这是我 Supabase 平台自己在处理，信我。”**

带着这把 key 去请求数据库时，**RLS 会被自动绕过**，就像 PostgreSQL 看到 root 用户操作一样，不再检查谁是谁。

---

### 🎯 场景举例（非常实用）：

| 场景                                    | 是否使用 Service Role Key |
| ------------------------------------- | --------------------- |
| 用户前端页面请求自己的数据                         | ❌ 不用（默认 RLS 生效）       |
| 后台 Server Action 写入当前用户的 `user_id` 字段 | ✅ 需要（否则你写不进去）         |
| 定时任务批量更新全体用户的状态                       | ✅ 需要                  |
| webhook 接收 Stripe 支付回调，写入付款信息到用户表     | ✅ 需要                  |
| 管理员后台查看所有用户订单                         | ✅ 需要                  |

---

### 🚨 安全重点提醒

* Service Role Key 就像数据库“超级管理员账号”
* **千万不能暴露在前端浏览器代码中！**
* 最安全的做法是：

  * `NEXT_PUBLIC_SUPABASE_ANON_KEY` 供前端使用（受 RLS 限制）
  * `SUPABASE_SERVICE_ROLE_KEY` 仅在 server action、API route 中使用（无限制）

---

### 🧠 你这句话最值得点赞：

> “RLS见到该 key 即放行” —— 完全正确 ✅

