# Firebase 登录示例

静态网页，Firebase Authentication 接入 Google / Facebook / Apple 三方登录。预留了 `window.AuthBridge` 接口层，方便以后接入 Unity WebGL。

## 目录

- [第一步：创建 Firebase 项目](#第一步创建-firebase-项目)
- [第二步：注册 Web 应用，拿到配置](#第二步注册-web-应用拿到配置)
- [第三步：开启 Google 登录（最简单，建议先做这个）](#第三步开启-google-登录)
- [第四步：部署到 GitHub Pages，并加入授权域名](#第四步部署到-github-pages并加入授权域名)
- [第五步（可选）：开启 Facebook 登录](#第五步可选开启-facebook-登录)
- [第六步（可选）：开启 Apple 登录](#第六步可选开启-apple-登录)
- [Unity 对接说明](#unity-对接说明)

---

## 第一步：创建 Firebase 项目

1. 打开 [console.firebase.google.com](https://console.firebase.google.com)，用你的 Google 账号登录
2. 点击「创建项目 / Add project」
3. 起个名字（比如 `firebase-auth-demo`），下一步
4. Google Analytics 这一步可以直接关掉（demo 项目不需要），点击「创建项目」
5. 等待几十秒，项目创建完成

## 第二步：注册 Web 应用，拿到配置

1. 进入项目后，点击首页中间的 `</>`（Web）图标，开始添加 Web 应用
2. 起个应用昵称（随意），**不要**勾选"同时为此应用设置 Firebase Hosting"（我们用 GitHub Pages，不需要）
3. 点击「注册应用」，会看到一段 `firebaseConfig` 代码，形如：
   ```js
   const firebaseConfig = {
     apiKey: "AIzaSy...",
     authDomain: "xxx.firebaseapp.com",
     projectId: "xxx",
     storageBucket: "xxx.appspot.com",
     messagingSenderId: "123456789",
     appId: "1:123456789:web:abcdef"
   };
   ```
4. 把这段配置复制下来，发给我，或者你自己打开 [`index.html`](./index.html) 替换掉里面同名的 `firebaseConfig` 对象

> 这些值不是密钥，Firebase Web 应用的配置本来就是设计成公开写在前端代码里的，真正的权限控制在后面的 Authentication 设置和数据库安全规则里，所以直接写进这个公开仓库没有安全问题。

## 第三步：开启 Google 登录

1. Firebase 控制台左侧菜单 → **Build / 构建 → Authentication**
2. 点击「开始使用 / Get started」
3. 「Sign-in method」标签页 → 找到 **Google** → 点击启用
4. 选一个"项目公开支持电子邮件地址"（一般用你自己的邮箱）→ 保存

到这一步 Google 登录已经可以用了（Facebook / Apple 需要额外的开发者账号配置，见下面可选步骤）。

## 第四步：部署到 GitHub Pages，并加入授权域名

1. 把配置好的 `index.html` 推送到这个仓库（我可以帮你做）
2. 仓库 Settings → Pages → Source 选 `Deploy from a branch`，Branch 选 `main` / `/(root)`
3. 拿到地址后（形如 `https://lijian316.github.io/firebase-auth-demo/`），回到 Firebase 控制台
4. **Authentication → Settings → 授权域名（Authorized domains）** → 添加：
   ```
   lijian316.github.io
   ```
   不加这一步，登录弹窗会报 `auth/unauthorized-domain` 错误。

做完这四步，打开你的 Pages 地址，点"使用 Google 继续"就能测试真实登录了。

## 第五步（可选）：开启 Facebook 登录

比 Google 麻烦一些，需要单独注册一个 Facebook 应用：

1. 打开 [developers.facebook.com](https://developers.facebook.com)，创建一个应用（类型选"消费者"或"其他"）
2. 应用后台 → 添加产品 → **Facebook 登录** → Web 设置
3. 「有效的 OAuth 重定向 URI」填入 Firebase 提供的回调地址（在 Firebase 控制台 Facebook 登录设置页面能看到，形如 `https://xxx.firebaseapp.com/__/auth/handler`）
4. Facebook 应用「设置 → 基本」里拿到 **应用编号（App ID）** 和 **应用密钥（App Secret）**
5. 回到 Firebase 控制台 → Authentication → Sign-in method → **Facebook** → 启用 → 把 App ID / App Secret 填进去 → 保存
6. Facebook 应用还需要在「应用审核」里申请 `public_profile`、`email` 权限才能正式上线给非测试用户使用（开发阶段用自己的 Facebook 账号加到"测试用户"里即可先用）

## 第六步（可选）：开启 Apple 登录

三个里最麻烦的一个，前提条件：

- 需要一个 **付费的 Apple Developer Program 账号**（每年 $99）
- 需要有自己的域名验证权限（GitHub Pages 的 `https://lijian316.github.io` 可以用）

大致步骤：

1. [developer.apple.com](https://developer.apple.com) → Certificates, Identifiers & Profiles
2. 创建一个 **Services ID**，配置 "Sign in with Apple"，填入你的域名和回调地址（Firebase 控制台 Apple 登录设置页会给你确切的回调地址）
3. 创建一个用于 Sign in with Apple 的 **私钥（Key）**，下载 `.p8` 文件，记住 Key ID 和 Team ID
4. 回到 Firebase 控制台 → Authentication → Sign-in method → **Apple** → 启用 → 把 Services ID、Team ID、Key ID、私钥内容填进去 → 保存

这一步涉及付费账号和证书管理，建议先把 Google（和可选的 Facebook）跑通，确认整个登录流程没问题之后，有需要再回来做 Apple。

## Unity 对接说明

页面里暴露了一个全局对象 `window.AuthBridge`，专门为以后接入 Unity WebGL 准备的：

```js
AuthBridge.signInWithGoogle()      // 返回 Promise，resolve 出用户信息
AuthBridge.signInWithFacebook()
AuthBridge.signInWithApple()
AuthBridge.signOut()
AuthBridge.getCurrentUser()        // 同步返回当前用户信息，未登录是 null
AuthBridge.onChange(callback)      // 登录状态变化时会调用 callback(userOrNull)
```

用户信息的字段固定是：

```js
{ uid, displayName, email, photoURL, providerId }
```

以后 Unity WebGL 项目里写一个 `.jslib` 插件，用 `mergeInto(LibraryManager.library, {...})` 把这些方法包一层暴露给 C# 调用，在 JS 侧的回调里用 `unityInstance.SendMessage(gameObjectName, methodName, JSON字符串)` 把结果传回 Unity 就行——目前这层已经和具体框架解耦，接入时不需要改这边的登录逻辑。
