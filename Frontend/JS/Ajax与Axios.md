# Ajax 与 Axios

## 1. Ajax

概念：**Asynchronous JavaScript And XML**，异步的 JavaScript 和 XML。

作用：

- **数据交换**：通过 Ajax 可以给服务器发送请求，并获取服务器响应的数据。
- **异步交互**：可以在**不重新加载整个页面**的情况下，与服务器交换数据并更新**部分网页**（如搜索联想、用户名是否可用的校验）。

> 名字里的 **XML**（Extensible Markup Language，可扩展标记语言）本质是一种**数据格式**，可以用来存储复杂的数据结构。
>
> 它是 Ajax 诞生时的数据载体，但现在前后端传输的数据基本都用 **JSON**（体积更小、JS 原生支持解析），XML 已很少见——所以 "Ajax" 这个名字其实是历史遗留，**不代表现在还在用 XML**。详见 [JavaScript基础](JavaScript基础.md) 的 JSON 一节。

## 2. 同步与异步

核心差别在于：请求发出到响应返回的这段时间里，**客户端能不能做其他事**。

|      | 同步                | 异步                |
| ---- | ----------------- | ----------------- |
| 请求发出后 | 客户端**等待**，期间不能做其他操作 | 客户端**可以执行其他操作**   |
| 响应返回后 | 才继续往下执行           | 由回调处理响应结果，页面整体不刷新 |

## 3. Axios

**Axios 对原生的 Ajax 进行了封装**，简化书写，快速开发。
### 3.1 原生 Ajax 的痛点

原生 Ajax 靠 `XMLHttpRequest` 对象实现，一个最简单的 GET 请求要写这么多：

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET', 'https://web-server.itheima.net/emps/list');  // 配置请求方式与地址
xhr.onload = () => {
  console.log(JSON.parse(xhr.responseText));  // 响应只是一段文本，要自己解析
};
xhr.onerror = () => { /* 自己处理失败 */ };
xhr.send();  // 发送请求
```

问题集中在三点：

| 痛点 | 具体表现 |
| --- | --- |
| 写法繁琐 | 配置、监听、发送分好几步，每发一个请求都要重复一遍 |
| 回调嵌套 | 多个请求有依赖关系时层层嵌套（回调地狱），流程难读难改 |
| 细节要自己兜 | 参数拼接、JSON 解析、状态码判断、超时，全部手动处理 |

**Axios 把上面这些都封装掉了**：内部仍是 `XMLHttpRequest`，但对外只暴露一个配置对象 + Promise，写法从"步骤"变成"声明"。

### 3.2 基本用法

使用步骤：引入 Axios 的 js 文件 → 使用 Axios 发送请求并获取响应结果。

```html
<!-- CDN 方式引入（也可以 npm 安装） -->
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>
```

```javascript
axios({
  method: 'GET',                                    // 请求方式
  url: 'https://web-server.itheima.net/emps/list',  // 请求路径
}).then((result) => {                               // 成功回调
  console.log(result.data);
}).catch((err) => {                                 // 失败回调
  alert(err);
});
```

> `data`（POST 的请求体数据）与 `params`（拼在 url 后的查询参数，如 `?key=val`）容易混淆，需要时询问 AI 或查官网即可。

## 4. 请求方式别名（推荐）

为了方便起见，Axios 已经为所有支持的请求方法提供了别名，格式为 `axios.请求方式(url [, data [, config]])`：

```javascript
axios.get('https://mock.apifox.cn/m1/xxx/emps/list').then((result) => {
  console.log(result.data);
}).catch((err) => {
  console.log(err);
});

axios.post('https://mock.apifox.cn/m1/xxx/emps/update', 'id=1').then((result) => {
  console.log(result.data);
}).catch((err) => {
  console.log(err);
});
```

## 5. async / await

用 `async`、`await` **可以让异步变为同步操作**：`async` 用来声明一个异步方法，`await` 用来等**待异步任务执行完**，并拿到结果。

```javascript
methods: {
  async search() {
    // 根据用户输入的搜索条件，基于 axios 发送异步请求到服务端
    let result = await axios.get('https://web-server.itheima.net/emps/list?name=xxx&gender=xxx&job=xxx');
    this.employees = result.data.data;
  }
}
```

与 3.2 节 `.then()` 写法的差别：

| | `.then()` 回调 | `async / await` |
| --- | --- | --- |
| 结果获取 | 只能在回调函数里拿 | `await` 直接返回结果值，赋给变量继续往下写 |
| 多步依赖 | 中间结果只能靠 `return` 一环环往下传 | 中间结果就是普通变量，**一行一步、可读性强、便于维护** |

> 注意：
> - `await` 关键字**只在 `async` 函数内有效**。

