---
id: API编排
---

API 编排功能可以通过组合的方式整合处理多个 API / 业务节点的数据，输出业务终端需要的数据结果

## API 编排入门

### 基础

API 编排本质上是一种用可视化的方式来编写后端代码，它可以用于不复杂的接口聚合及增删改查场景。

API 编排是树形接口，它的执行顺序是从上至下执行，比如下图：

![tree](/img/API/API编排/flow-tree.png)

这个 API 编排首先运行「并行执行」节点，这里面的子节点会并行运行，接着是循环节点，可以对前面的输出结果进行循环处理，然后在这个循环里面有个分支判断，如果成功就运行「设置变量」的节点，最后再运行「发送邮件」节点，如果熟悉写代码，上面这个树等价于下面的代码：

```javascript
// 这个是为了处理并行执行
Promise.all([
  httpResult = await sendHTTPAction(...),
  sqlResult = await runSQL(...)
]);
// 对 http 返回结果进行二次处理
for (const item of httpResult) {
  // 分支
  if (item.status == 1) {
    // 设置变量
    item.status = '成功';
  }
}
// 发送邮件
sendMail(...)
```

各个节点之间如何协作？方法是通过变量，比如发起一个 HTTP 接口，其中有个配置是「存入变量」

![var](/img/API/API编排/var.png)

执行完后，下面的节点就能通过这个变量来获取到内容，比如后面接一个「设置变量」的节点，在这个节点中，可以通过 `{{}}` 来获取到，比如下面这个例子获取到前面 output 变量中的 name 值，然后又将它写入变量 res。

![var-set](/img/API/API编排/var-set.png)

当所有节点执行完后，output 变量的值就作为最终结果返回前端。

### 创建和编辑

在【API 中心】列表中点击【新增 API】，弹出面板选择【多接口聚合模式】，即可新增一个 API 编排

![new](/img/API/API编排/new.png)

保存后在列表中点击【多接口聚合设计】，即可进行整个 API 编排的设计

![merge](/img/API/API编排/merge.png)

点击连线上的 + 号，可添加连线所在层级的平级节点，点击节点右侧的 + 号，可添加该节点的子节点
![add](/img/API/API编排/add.png)

### 一个简单的例子

本例子将实现一个从服务端获取文章列表并对日期进行格式化后返回到前端的 API 编排

添加一个 “发送 HTTP 请求” 类型的节点，在 “请求地址” 输入框中填写以下 url

`https://3xsw4ap8wah59.cfc-execute.bj.baidubce.com/apiArrangeDemo/list`

![http](/img/API/API编排/http.png)

点击工具栏中的 “调试接口” 按钮
![run](/img/API/API编排/run.png)

在弹出框中点击右下角的 “发送请求” 按钮，等待请求执行完毕后，可以看到请求结果和运行日志

![send](/img/API/API编排/send.png)

可以看到当前返回的数据是包含 items 和 total 两个字段的 json 格式数据，items 列表单项中的 date 字段是以毫秒时间戳格式返回的日期

```
{
  "items": [
    {
      "id": 1,
      "title": "标题0",
      "subscribe": "摘要0",
      "date": 1622793580560
    },
    ...
  ],
  "total": 200
}
```

这里我们希望将 date 字段转换为能够友好展示的日期字符串

为 “发送 HTTP 请求” 节点添加一个平级 “循环” 类型的节点，并为 “循环” 节点添加一个 “日期格式化” 子节点

“循环” 节点配置如下，接口返回的数据会被统一放在全局的 output 对象中，通过 `output.xxx` 可以取到相关字段
![loop](/img/API/API编排/loop.png)

“日期格式化” 节点配置如下，循环节点的行为是循环执行自身子节点的逻辑
![dateformat](/img/API/API编排/dateformat.png)

点击发送请求，可以看到返回数据中的 date 被格式化成了 `YYYY-MM-DD HH:mm:ss` 格式

```
{
  "items": [
    {
      "id": 1,
      "title": "标题0",
      "subscribe": "摘要0",
      "date": "2021-06-04 17:23:41"
    },
    ...
  ],
  "total": 200
}
```

调试完之后需要点右下角的 “确认” 按钮保存，编辑界面中的内容才会被保存到数据库中
关于如何在页面中使用 API 编排获取数据，请看下一节

### 在页面中使用 API 编排

在页面中使用 API 编排的方式，和使用 API 中心 API 的方式完全相同，下面同样以一个简单的例子来说明

新建一个普通页面，在页面中拖入一个增删改查组件

勾选增删改查组件的数据拉取接口的 “API 中心” 选项

![select-apicenter](/img/API/API编排/select-apicenter.png)

然后点选输入框右侧的按钮，显示 API 中心列表弹框

![select-apicenter-btn](/img/API/API编排/select-apicenter-btn.png)

在列表中选择之前创建的 API 编排（和单个接口 API 在同一个列表中）

![select-apicenter-dialog](/img/API/API编排/select-apicenter-dialog.png)

配置字段映射，可以看到 API 编排处理返回的数据被正常展示在列表中

![crud](/img/API/API编排/crud.png)

### 调试接口

点击顶部的「调试接口」按钮进入接口调试功能。

![debug](/img/API/API编排/debug.png)

在接口调试弹窗中，可以输入提交参数，然后点击「发送请求」进行测试，除了查看输出结果，最有用的功能是查看执行过程的记录，如下图所示：

![debug-result](/img/API/API编排/debug-result.png)

对于初始节点和有数据变更的节点，还能点击旁边的加号来查看当时的数据情况，这样就能知道每一步操作是否正确对数据进行了处理。

![debug-state](/img/API/API编排/debug-state.png)

### 查看配置源码

点击顶部的「查看配置源码」可以看到最终生成的树形结构配置，这是 API 编排在实际运行是所使用的格式，这个功能主要是用于结果预期不一致时进行调试，可以将这个配置发给爱速搭的管理员协助分析。

![config-source](/img/API/API编排/config-source.png)

### 生成的伪代码示例

点击顶部的「生成的伪代码示例」可以生成伪代码示例。

![pseudocode](/img/API/API编排/pseudocode.png)

这个功能主要是给习惯看源码的研发，可以方便看出执行流程，不过需要注意的是爱速搭实际执行并不是使用这个生成的代码，因此这里仅供参考，输出的伪代码并不保证能运行。

## 节点分类介绍

接下来详细介绍各个节点的功能

### 数据接口

#### 发送 HTTP 请求

最基本的 HTTP 请求发送节点，使用方法同【单个接口 API】，具体见[API 中心](./API中心.md)

#### 数据源 SQL

在数据源上执行 SQL 操作。

在 SQL 语句里用 `{{}}` 包裹的部分可以执行 JS 语句，实现灵活的处理，比如：

```
select * from blog where title = {{ input.title }}
```

它的实现原理是最终转成 `select * from blog where title = ?`，然后将 `input.title` 的值作为后续参数填入，因此默认会进行变量转义，防止 sql 注入

如果数据是一维数组，使用如下写法

```
select * from blog where title in ({{ input.titles}})
```

如果是二维数组，比如下面的 `input.blogs` 是 `[['标题1', '内容1'], ['标题2', '内容2']]`，使用如下写法

```
INSERT INTO blog VALUES {{ input.blogs }}
```

或者指明字段

```
INSERT INTO blog (title, content) VALUES {{ input.blogs }}
```

如果数据是对象数组形式，需要先转成二维数组，可以用 js 节点操作，比如

```
module.exports = async function (event, state) {
  state.blogs = [state.input.blogs.map(item => [item.title, item.content])];
  return state;
};
```

如果要实现有参数时才查询，可以用如下写法

```
select * from blog where ( {{ !input.title }} OR title = {{ input.title }} )
```

这个语句在输入参数有 `title` 的时候，会变成

```
select * from blog where (FALSE OR title = ?)
```

如果输入参数里没有 `title`，则会变成下面的语句，可以看到前面的条件是 true，所以 or 里的条件不会生效。

```
select * from blog where (TRUE OR title = ?)
```

如果输出的内容是 sql 关键字或列名，不能使用之前的语法，比如

```sql
# 这个写法的错误的
select * from blog order by {{title}}
```

原因是这个写法会变成 `select * from blog order by ?`，而通过变量替换后，就变成了 `select * from blog order by "title"`

如果要输出关键字或列名需要使用 `#{{}}` 来包裹

```sql
select * from blog order by #{{title}}
```

在这种写法下会原样输出内容（会滤掉分号、双引号、逗号等特殊字符，但不会去掉空格），如果 title 的内容是 a，输出结果就是

```
select * from blog order by a
```

如果 title 内容是个数组，比如 `['a', 'b']`，输出结果将会是

```
select * from blog order by a, b
```

因为它会去掉引号，所以不能直接用在变量的场景，比如

```
select * from blog where title = #{{query.title}}
```

需要自己加引号，但不推荐用这个语法来实现动态内容，下面的场景最好还是用 `{{}}`

```
select * from blog where title = "#{{query.title}}"
```

### 执行控制

#### 分支

相当于代码中的 `if` 语句。

#### 循环

循环用于对数组进行遍历，它的每个子节点用于执行每次循环的中的语句。

#### 跳出循环

用于跳出循环，不再执行循环的后续所有操作，相当于代码中的 `break` 语句。

#### 继续循环

用于跳过当前循环的后续操作，进入下一个循环，相当于代码中的 `continue` 语句。

#### 退出

用于退出整个接口，后续不再执行任何操作

### 并行/串行

#### 顺序执行

按顺序执行子节点

#### 并行执行

这个节点下的子节点会并行执行，用于独立获取各种数据的场景。

### 数据处理

#### 设置变量

赋值语句，用于对变量进行赋值。

#### 日期格式化

对日期格式进行转换，比如将时间戳转成可读的日期格式。

#### JS 代码

通过 js 代码的方式来对数据进行处理，用于处理一些比较复杂的数据转换场景。

最简单的示例如下

```javascript
module.exports = async function (event, state) {
  return state;
};
```

其中 state 是 api 编排中的全局数据，比如前面如果有个 HTTP 节点将返回结果「存入变量」到 data 下，在 JS 代码中就能通过 `state.data` 获取到这个值，然后进行二次转换。

同时还能写入 `state.output` 来影响 api 聚合最终的输出结果。

这段 JS 函数的返回值会替换全局数据，因此上面的例子中直接返回 `state`。

### 其他

#### 发送邮件

用于发送邮件，比如进行邮件通知

#### session

有一个特殊的全局变量 session，它会在整个用户会话中存储，如果写入这个变量，下次再次请求的时候还能读取到。

这个变量的值可以是字符串，也可以是对象，但整体大小限制在 2k 字节。

session 是应用级别隔离，不同应用在不同环境下是独立的。
