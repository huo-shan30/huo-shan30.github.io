# HTTP笔记

## 一、HTTP 是什么

- **全称**：超文本传输协议（HyperText Transfer Protocol）
- **定位**：应用层协议，专门用来在客户端与服务器之间传输超媒体文档（HTML、图片、脚本等）。
- **模型**：**客户端—服务器**模型。一次请求，一次响应。
- **核心特征**
  - **无状态**：每个请求都是独立的，服务器处理完就忘。为了模拟连续会话（如购物车），需要额外机制（Cookie、Token）。
  - **可扩展**：通过新增方法、状态码、头部字段，可以不断给 HTTP 增加能力，就像给同一框架不断加插件。
  - **基于可靠传输**：传统上依赖 TCP（保证数据不丢、不乱序），HTTP/3 改用 QUIC（基于 UDP）增强性能。

## 二、系统组成

- **客户端（用户代理）**  
  通常是浏览器，也可以是爬虫或调试工具。客户端永远第一个发言，先发请求。浏览器会解析 HTML，然后像多线程下载一样，自动发出很多子请求去取 CSS、JS、图片。

- **服务器（Web 服务器）**  
  可以是单台机器，也可以是背后一堆机器的集群（负载均衡）。同一个 IP 上能通过 `Host` 头区分不同网站（虚拟主机），就像一栋大楼里通过房间号找到不同公司。

- **代理（Proxy）**  
  站在客户端和服务器之间的“中间人”，常见作用：
  - 正向代理：帮你访问外网（比如在公司上网）。
  - 反向代理：站在服务器前面，负责负载均衡、缓存、SSL 终结，对客户端是透明的。
  - 其他功能：内容过滤、安全扫描、日志记录等。

## 三、HTTP 能控制什么

- **缓存**：用头部告诉浏览器/中间代理“这个资源可以存多久，什么时候要重新验证”。
- **解除同源限制**：浏览器默认禁止跨站读取数据（同源策略），服务器通过 CORS 头部（如 `Access-Control-Allow-Origin`）说“我允许来自其他域名的请求”，从而实现跨域。
- **认证**：通过 `401` + `WWW-Authenticate` 要求登录，或通过 `Authorization` 头携带 Token，或通过 Cookie 保持会话。
- **会话**：HTTP 无状态，但可以用 Cookie 让多次请求挂上同一个会话 ID，或者用 Token（JWT）在请求里自己证明身份，这样就能实现“登录一次，后续自动识别”。


## 四、 URL 与 资源定位

一个典型的 URL 结构：
```
https://www.example.com:443/path?query=1#fragment
|------| |---------------| |--| |------| |------| |------|
 协议     主机（域名）     端口   路径    查询参数   片段
```
HTTP 协议通过 URL 来定位互联网上的任意资源。

## 五、 报文结构

HTTP 报文分为**请求报文**和**响应报文**，结构类似（HTTP/1.1 格式）：

**请求报文**：
```
GET /index.html HTTP/1.1         ← 请求行（方法、URI、协议版本）
Host: www.example.com            ← 请求头部（Header）
User-Agent: Mozilla/5.0
Accept: text/html
                                 ← 空行（CRLF）
                                 ← 可选的消息主体（Body）
```

**响应报文**：
```
HTTP/1.1 200 OK                  ← 状态行（版本、状态码、原因短语）
Content-Type: text/html; charset=utf-8
Content-Length: 1234
                                 ← 空行
<!DOCTYPE html>...               ← 消息主体
```

## 六、 请求方法

定义对资源的操作意图，常见方法：

| 方法 | 含义 | 安全性（不修改） | 幂等性（多次执行结果相同） |
|------|------|:-----------:|:------------:|
| **GET** | 获取资源 | ✅ | ✅ |
| **POST** | 提交数据（创建/更新） | ❌ | ❌ 一般不是 |
| **PUT** | 完整替换资源 | ❌ | ✅ |
| **DELETE** | 删除资源 | ❌ | ✅ |
| **HEAD** | 只获取响应头 | ✅ | ✅ |
| **OPTIONS** | 查询服务器支持的方法 | ✅ | ✅ |
| **PATCH** | 部分修改资源 | ❌ | 不保证 |

## 七、 状态码

服务器用三位数字告诉客户端请求处理的结果：

- **1xx 信息性**：临时响应，如 `100 Continue`
- **2xx 成功**
  - `200 OK` — 请求成功
  - `201 Created` — 新资源已创建
  - `204 No Content` — 成功但无返回主体
- **3xx 重定向**
  - `301 Moved Permanently` — 永久跳转（缓存新地址）
  - `302 Found` — 临时跳转
  - `304 Not Modified` — 资源未修改，可使用缓存
- **4xx 客户端错误**
  - `400 Bad Request` — 请求语法错误
  - `401 Unauthorized` — 需身份认证
  - `403 Forbidden` — 无权限访问
  - `404 Not Found` — 资源不存在
  - `405 Method Not Allowed` — 方法不允许
- **5xx 服务端错误**
  - `500 Internal Server Error` — 服务器内部错误
  - `502 Bad Gateway` — 上游服务器无效响应
  - `503 Service Unavailable` — 暂时过载或维护

## 八、 头部（Header）

头部携带元数据，可分为**通用头、请求头、响应头、实体头**。关键例子：

- **Host**： 标识服务器域名（HTTP/1.1 必须）
- **User-Agent**： 客户端软件信息
- **Accept** / **Content-Type**： 协商数据格式，如 `application/json`、`text/html`
- **Content-Length** / **Transfer-Encoding**： 指示正文长度或分块传输
- **Cache-Control**： 控制缓存行为（强缓存/协商缓存）
- **Cookie** / **Set-Cookie**： 维持会话状态
- **Authorization**： 携带认证凭证（如 Bearer token）
- **CORS 相关头**： `Access-Control-Allow-Origin` 等跨域控制
重要头部解释：
- `Host`：必带，告诉服务器我要访问哪个网站，同 IP 多个站点靠它区分。
- `Transfer-Encoding: chunked`：当内容长度事先不知道时，分块传输，边生成边发送。
- `ETag` / `Last-Modified`：用于缓存验证，像资源的“指纹”或“最后修改时间”。

## 九、 如何解决“无状态”问题

HTTP 本身不保留用户状态，常用手段让服务端识别连续请求：

- **Cookie + Session**：服务器把 Session ID 放入 `Set-Cookie`，客户端后续回传。
- **Token（JWT 等）**： 客户端存储令牌，每次通过 `Authorization` 头发送。
- **URL 重写**： 将标识符直接拼在 URL 中（现在很少用）。

## 十、 缓存机制

提升性能、减少带宽的关键：

- **强缓存**：`Expires`（绝对时间）或 `Cache-Control: max-age=3600`，有效期内直接使用本地缓存，不发请求。
- **协商缓存**：
  - 服务器返回 `Last-Modified` / `ETag`
  - 客户端再次请求携带 `If-Modified-Since` / `If-None-Match`
  - 服务器比对后，若没变化返回 `304 Not Modified`，客户端用本地缓存；有变化返回 `200` 及新数据。

## 十一、 HTTPS（安全传输）

HTTPS = HTTP + TLS（传输层安全），默认端口443，提供：

- **加密**：防窃听
- **身份认证**： 用证书确保服务器真伪
- **数据完整性**： 防篡改

TLS 握手协商密钥后，所有 HTTP 报文都被加密传输。现代浏览器逐渐要求 HTTPS，HTTP/2 和 HTTP/3 更是要求必须使用 HTTPS。

## 十二、 版本演进

| 版本 | 发布时间 | 核心特性 |
|------|----------|----------|
| HTTP/0.9 | 1991 | 仅支持 GET，无头部，返回 HTML |
| HTTP/1.0 | 1996 | 增加头部、POST/HEAD、状态码，短连接 |
| **HTTP/1.1** | 1997~1999 | **持久连接**（Keep-Alive）、管道化、Host 头、分块传输、缓存改进 |
| HTTP/2 | 2015 | **多路复用**、头部压缩（HPACK）、服务器推送（Server Push）、二进制分帧 |
| **HTTP/3** | 2022 (标准化) | 基于 **QUIC/UDP**，零 RTT 连接建立、无队头阻塞、内置加密，移动网络体验更佳 |

目前 HTTP/1.1 仍是兼容基础，HTTP/2 和 HTTP/3 已大规模普及，尤其 APP 和现代浏览器默认使用最新协议。

## 十三、 一个完整交互流程简述

1. **DNS 解析**：域名 → IP 地址
2. **建立连接**：TCP 三次握手（若 HTTP/3 则是 QUIC 握手）
3. **TLS 握手**（HTTPS 时）：协商加密套件、验证证书、生成会话密钥
4. **发送请求**：客户端构造请求报文并发送
5. **服务器处理**：路由、业务逻辑、生成响应
6. **返回响应**：状态码、头部、主体返回
7. **客户端渲染/处理**：浏览器解析 HTML/CSS/JS，可能发起额外请求
8. **连接管理**：HTTP/1.1 默认保持一段时间以便复用，HTTP/2/3 多路复用更高效

---

你的 PHP 环境已经就绪，接下来要掌握的就是编写代码的语法和规则。PHP 的基础很平易近人，下面我会用**示例+解释**的方式，带你快速入门。

---

# PHP笔记

## 1. PHP 代码写在哪里？
PHP 代码必须放在 `<?php` 和 `?>` 标签内，文件扩展名通常是 `.php`。
```php
<?php
  echo "Hello, PHP!";
?>
```
如果文件是纯PHP代码，结尾的 `?>` 可以省略（这是最佳实践，避免意外输出空格）。

---

## 2. 变量与数据类型

- **变量**：以 `$` 开头，大小写敏感。
- **弱类型**：PHP 会自动转换类型。
- **常用类型**：
  - 字符串 `string`、整型 `int`、浮点 `float`、布尔 `bool`
  - 数组 `array`、对象 `object`、`NULL`
  - 用 `var_dump($var)` 可以查看类型和值。

```php
<?php
$name = "小明";          // 字符串
$age  = 25;             // 整数
$height = 1.75;         // 浮点数
$is_student = true;     // 布尔值
echo "我叫{$name}，今年{$age}岁。";
```
> 💡 双引号字符串内可直接解析变量，单引号则视为纯文本。

---

## 3. 字符串拼接与输出
- `echo`：输出一个或多个字符串（无返回值，速度快）。
- `print`：类似echo，但返回值 1，只能输出一个字符串。
- 拼接符：`.`（点号）

```php
echo "Hello " . $name . "!";   // Hello 小明!
```

---

## 4. 运算符

**算术**：`+`, `-`, `*`, `/`, `%`（取模）, `**`（乘方）  
**比较**：`==`, `===`（全等，类型也相同）, `!=`, `!==`, `>`, `<`, `>=`, `<=`  
**逻辑**：`&&` (and), `||` (or), `!` (not)  
**三元运算符**：`$result = ($age >= 18) ? '成年' : '未成年';`  
**空合并**（PHP 7+）：`$name = $_GET['name'] ?? '默认名';`（若变量存在且不为NULL则用，否则用默认值）

---

## 5. 条件判断
**if-elseif-else**：
```php
$score = 85;
if ($score >= 90) {
    echo "优秀";
} elseif ($score >= 60) {
    echo "及格";
} else {
    echo "不及格";
}
```

**switch**：
```php
$fruit = "apple";
switch ($fruit) {
    case "apple":
        echo "苹果";
        break;
    case "banana":
        echo "香蕉";
        break;
    default:
        echo "其他水果";
}
```

---

## 6. 循环
**while** / **do-while** / **for** / **foreach**（专门遍历数组）

```php
// for 循环
for ($i = 0; $i < 5; $i++) {
    echo $i . " ";
}

// foreach 遍历数组
$colors = ["红", "绿", "蓝"];
foreach ($colors as $color) {
    echo $color . " ";
}
// 获取键名和值
foreach ($colors as $key => $value) {
    echo "索引{$key}：{$value} ";
}
```

---

## 7. 函数
使用 `function` 关键字定义，可以传参、返回值。
```php
function greet($name = "朋友") {   // 默认值
    return "你好，" . $name;
}
echo greet();          // 你好，朋友
echo greet("小明");    // 你好，小明
```

---

## 8. 数组（极其重要）
PHP 的数组非常强大，本质上是有序映射。

**创建数组**：
```php
// 索引数组
$fruits = ["苹果", "香蕉", "橘子"];   // 或 array(...)

// 关联数组（类似其他语言的字典/Map）
$user = [
    "name" => "张三",
    "age"  => 28
];

// 多维数组
$users = [
    ["name" => "张三", "age" => 28]
];
```

**常用数组操作**：
```php
$arr = [1, 2, 3];
array_push($arr, 4);       // 末尾添加
count($arr);               // 长度
in_array(2, $arr);         // 检查值是否存在
array_merge($arr1, $arr2); // 合并数组
// 更多：array_keys, array_values, array_filter, array_map...
```

---

## 9. 表单与常用超全局变量
PHP 预定义了 `$_GET`, `$_POST`, `$_REQUEST`, `$_SERVER`, `$_SESSION`, `$_COOKIE` 等变量，用于获取请求数据。

**示例：处理用户登录表单**
```html
<!-- login.html -->
<form action="login.php" method="post">
    用户名: <input type="text" name="username">
    密码: <input type="password" name="password">
    <button type="submit">登录</button>
</form>
```
```php
// login.php
<?php
$username = $_POST['username'] ?? '';
$password = $_POST['password'] ?? '';

if ($username === 'admin' && $password === '123456') {
    echo "登录成功！";
} else {
    echo "用户名或密码错误";
}
```
> 实际开发中务必过滤和验证输入，并用**预处理语句**操作数据库，防止SQL注入。

---

## 10. 包含其他文件
`include`/`require`：把别的 PHP 文件引入当前文件。
- `include`：文件不存在时发出警告（E_WARNING），脚本继续运行。
- `require`：文件不存在时致命错误（E_ERROR），脚本停止。
- `include_once` / `require_once`：只包含一次。

```php
require 'config.php';    // 数据库配置
include 'header.php';    // 页头
echo "主要内容";
```

---

# RCE笔记

## 一、RCE 的分类
在 PHP 中，依据执行内容的不同，RCE 主要分两类：

| 类型 | 执行对象 | 典型危险函数（PHP） |
|------|----------|---------------------|
| **代码执行** | 执行任意 PHP 代码 | `eval()`、`assert()`（旧版）、`preg_replace()` 配合 `/e` 修饰符、`create_function()`、`call_user_func()` 等动态函数调用 |
| **命令执行** | 执行系统命令（Shell Command） | `system()`、`exec()`、`shell_exec()`、`passthru()`、`popen()`、`proc_open()`，以及**反引号** `` ` `` |

无论哪一种，只要攻击者的输入能够进入这些执行点，就可能导致服务器被完全控制。

---

## 二、代码执行示例（eval 的致命用法）
假设你写了这样一段“动态运算”的 PHP：
```php
<?php
$user_input = $_GET['expr'];   // 用户通过 URL 输入
eval("echo $user_input;");
```
正常情况下，用户访问 `http://host/index.php?expr=1+2` 会输出 `3`。

但攻击者可以传入恶意代码：
```
http://host/index.php?expr=1;phpinfo();
```
由于分号的作用，`eval` 实际执行了：
```php
echo 1;
phpinfo();   // 输出服务器配置，窥探环境
```
更危险的是：
```
http://host/index.php?expr=1;system('whoami');
```
这样就能直接执行系统命令，查看当前用户。如果权限够高，甚至能写入 Webshell、反弹 Shell。

> `assert()` 在 PHP < 7.2 时与 `eval` 行为类似，也属于高危函数。

---

## 三、命令执行示例（直接调用系统命令）
```php
<?php
$ip = $_GET['ip'];
system("ping -c 1 " . $ip);   // 拼接用户输入到系统命令中
```
正常使用：`http://host/ping.php?ip=127.0.0.1` 会执行 `ping`。

但是攻击者可以注入其他命令：
```
http://host/ping.php?ip=127.0.0.1;cat /etc/passwd
```
最终执行的命令变为：
```bash
ping -c 1 127.0.0.1;cat /etc/passwd
```
Linux 分号可分隔多条命令，所以 `cat /etc/passwd` 也会被执行，信息泄露。

此外，常见的命令连接符还有 `&&`、`||`、管道 `|`、反引号\`\`等，都可以用来构造复杂的多步攻击。

---

## 四、常见漏洞成因
1. **直接使用危险函数，且参数可控**  
   如上面例子，用户输入的 `$_GET`、`$_POST`、`$_COOKIE` 直接拼接到 `eval` 或 `system` 中。

2. **动态函数调用**  
   ```php
   $func = $_GET['action'];   // 攻击者传入 'system'
   $arg  = $_GET['cmd'];      // 传入 'whoami'
   call_user_func($func, $arg);  // 相当于执行 system('whoami')
   ```

3. **文件包含漏洞**有时也能变相 RCE（如通过 `include` 包含图片马、日志文件等），但在分类上通常单独列为文件包含漏洞。

4. **反序列化漏洞**（如 `unserialize`）可能触发魔术方法执行任意代码，也算 RCE 的一种复杂形式。

---

## 五、如何防御 RCE
核心原则：**永远不要信任用户输入，永远不要将用户输入直接拼接到可执行的上下文里。**

1. **避免使用危险函数**  
   - 能用其他安全方式实现的功能，坚决不用 `eval` / `system` 等。  
   - 若必须执行命令，使用参数数组形式（如 `proc_open` 配合数组参数）并严格限制可执行命令。

2. **严格过滤与白名单验证**  
   - 对数字、IP、域名等，按格式用 `filter_var` 或正则做白名单验证，不匹配则拒绝。  
   - 不要用黑名单，攻击者绕过的办法太多。

3. **使用更安全的替代函数**  
   - 代替 `eval`：如果只是解析简单表达式，可以考虑 `Symfony ExpressionLanguage` 等沙箱方案。  
   - 代替命令行 ping：使用 PHP 内置的 `socket` 或调用系统 API 库，而不是执行系统命令。

4. **环境与权限控制**  
   - `php.ini` 中设置 `disable_functions = eval,assert,system,exec,...` 禁用高危函数（但需注意不影响正常业务）。  
   - 运行 PHP 的 Web 用户（如 `www-data`）只给予最小权限，禁止执行 `/bin/sh` 等。

5. **使用 PHP 最新版本并更新**  
   旧版 PHP 的某些函数（如 `assert`、`create_function`）本身有危险特性，升级后要么被移除，要么更安全。

6. **部署 WAF（Web 应用防火墙）**  
   可以拦截包含恶意特征（如 `;cat`、`|whoami`）的请求，但不能完全依赖，只是额外防线。

---

