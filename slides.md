---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: prism
# show line numbers in code blocks
lineNumbers: true
# some information about the slides, markdown enabled
info: |
  ## 前端 Mock
# persist drawings in exports and build
drawings:
  persist: false
# download
download: true
exportFilename: mock
# use UnoCSS
css: unocss
---

# 前端 Mock

赵伟

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# 目录

- Mock 的作用
- Mock 的评价标准
- JSON-Server
- Mirage
- Mock Service Worker
- 总结


---

# Mock 的作用

<v-clicks>

- <material-symbols-rocket-launch-rounded style="color: red" /> 加快前端开发
-  <game-icons-octopus style="color: red" /> 让自测更全面更充分

</v-clicks>

---

# Mock 的作用

<br />

<MockImage />

---

# Mock 的评价标准

<v-clicks>

- 无缝衔接
- 简单易用

</v-clicks>

---

# JSON-Server

<br />

`JSON-Server` 使用 [`Express`](http://expressjs.com/) 开启一个后台服务。

---

# JSON-Server - JSON
<br />

1. 创建 `db.json` 文件
```json
{
  "posts": [
    { "id": 1, "title": "json-server", "author": "typicode" },
    { "id": 2, "title": "mirage", "author": "miragejs" }
  ]
}
```

<v-click>

2. 开启服务
```sh
$ json-server --watch db.json ---port 3000
```

</v-click>

---

# JSON-Server - JSON
<br />

自动生成请求路由
```js
GET     /posts     // 列表
GET     /posts/1   // 详情
POST    /posts     // 添加
PUT     /posts/1   // 全量替换
PATCH   /posts/1   // 局部更新
DELETE  /posts/1   // 删除
```

<v-click>

`get` `http://localhost:3000/posts/1`

```json
{ "id": 1, "title": "json-server", "author": "typicode" }
```

</v-click>

---
layout: title-two-cols
---

# JSON-Server - JSON

::left::
**优点**
<v-clicks>

- 简单
- 新增、修改、删除能自动同步数据
- 持久化数据
- 支持过滤、分页、排序等功能
- 可以用于非 web 应用，比如原生的应用

</v-clicks>

::right::

**缺点**
<v-clicks>

- 自动生成的请求路由可能不匹配后台 API
- 返回的数据格式可能不匹配后台 API
- 数据模型的主键只支持 `id`
- 数据硬编码

</v-clicks>

---

# JSON-Server - Module

<br />
创建 Mock 服务

```js{all|2}
const jsonServer = require("json-server");
const server = jsonServer.create();
const middlewares = jsonServer.defaults({ bodyParser: true });
server.use(middlewares);

// 自定义请求路由，自定义响应数据

server.listen(3000, () => {
  console.log("JSON Server is running");
});
```

---

# JSON-Server - Module

<br />
自定义请求路由，自定义响应数据

```js{all|11-13|12|1|3-9|4-8|15-19}
const users = ...;

function transform(data) {
  return {
    success: true,
    data: data,
    message: "success"
  };
}

server.post("/user/list", (req, res) => {
  res.send(transform(users));
});

server.post("/user/add", (req, res) => {
  const user = ...;
  users.push(user);
  res.send(transform(user));
});
```

---
layout: title-two-cols
---

# JSON-Server - Module

::left::
**优点**
<v-clicks>

- 可以自定义请求路由
- 可以自定义返回数据
- 生成假数据方便

</v-clicks>

::right::

**缺点**
<v-clicks>

- 新增、修改、删除需要自己同步数据
- 不支持自动更新，添加请求路由之后需要重启服务，可以使用 **nodemon** 工具实现自动更新
- 不支持浏览器调试

</v-clicks>

---

# Mirage

<br />

`Mirage` 重写了系统的 `fetch` 和 `XMLHttpRequest` 相关接口请求的方法。当我们在使用 `fetch` 和 `XMLHttpRequest` 请求数据的时候，其实调用的是 `Mirage` 重写的方法。

---

# Mirage

<br />
1. 创建 Mock 服务

```js{all|5-9}
import { createServer } from "miragejs"

export default function () {
  createServer({
    models: {},
    seeds(server) {},
    factories: {},
    routes() {},
    serializers: {}
  })
}
```

2. 在入口文件开启服务

```js
if (isMock()) {
  const makeServer = require("./mocks/index.js").default;
  makeServer();
}
```

---

# Mirage - Models


```js{2-7}
createServer({
    models: {
      user: Model.extend({
        org: belongsTo("organization"),
      }),
      organization: Model
    },
    seeds(server) {},
    factories: {},
    routes() {},
    serializers: {}
  })
```

---

# Mirage - Seeds

```js{3-6}
createServer({
    models: {},
    seeds(server) {
      server.create("user", { name: "张三" })
      server.create("user", { name: "李四" })
    },
    factories: {},
    routes() {},
    serializers: {}
  })
```

<v-click>

产生初始化数据

```json
[
  { "id": 1, "name": "张三" },
  { "id": 2, "name": "李四" }
]
```

</v-click>

---

# Mirage - Factories

```js{7-14|3-6|4|5}
createServer({
    models: {},
    seeds(server) {
      server.create("user");
      server.createList("user", 20);
    },
    factories: {
      user: Factory.extend({
        name: faker.name.findName,
        age: () => faker.datatype.number({ min: 18, max: 135 }),
        city: faker.address.cityName,
        createdAt: faker.date.past
      })
    },
    routes() {},
    serializers: {}
  })
```

---

# Mirage - Routes

```js{all|3-5|4|6-9|8|10-13|12|14-19|17|19-22|21}
createServer({
    routes() {
      this.get('/users', (schema) => {
        return schema.users.all()
      });
      this.get("/users/:id", (schema, request) => {
        const id = request.params.id
        return schema.users.find(id)
      })
      this.post("/users", (schema, request) => {
        const attrs = JSON.parse(request.requestBody)
        return schema.users.create(attrs)
      })
      this.patch("/users/:id", (schema, request) => {
        const newAttrs = JSON.parse(request.requestBody)
        const id = request.params.id
        return schema.users.find(id).update(newAttrs)
      })
      this.delete("/users/:id", (schema, request) => {
        const id = request.params.id
        return schema.users.find(id).destroy()
      })
    }
  })
```

---

# Mirage - Serializers

```js{1-10,17-19|4-8}
const ApplicationSerializer = Serializer.extend({
  serialize() {
    let json = Serializer.prototype.serialize.apply(this, arguments);
    return {
      success: true,
      data: Object.values(json)[0],
      message: "success"
    };
  }
});

createServer({
    models: {},
    seeds(server) {},
    factories: {},
    routes() {},
    serializers: {
      application: ApplicationSerializer
    }
  })
```

---
layout: title-two-cols
---

# Mirage

::left::
**优点**
<v-clicks>

- 结构清晰
- 很好地处理了数据模型之间的关系
- 生成和处理数据很方便
- 支持热更新，支持浏览器调试

</v-clicks>

::right::

**缺点**
<v-clicks>

- 在 `Network` 面板上 **看不到** 发出的请求，只能通过 `Console` 日志查看 request、response
- 数据模型的主键只支持 `id`

</v-clicks>

---

# Mock Service Worker (MSW)

<br />

Mock Service Worker 通过使用 [Service Worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) 来拦截实际的网络请求，提供自定义数据。

---

# Mock Service Worker

<br />
1. 创建 Mock 服务

```js{all|3-9}
import { rest, setupWorker } from 'msw';

const handlers = [
  rest.get('/user', (req, res, ctx) => {
     return res(
      ctx.json({ id: 1, name: "张三" })
    )
  })
]

export const worker = setupWorker(...handlers);
```

2. 在入口文件开启服务

```js
if (isMock()) {
  const { worker } = require('./mocks/index.js');
  worker.start();
}
```

---

# Mock Service Worker

```js{all|5-18|12|7|22|21-23}
import { factory, primaryKey } from "@mswjs/data";
import { faker } from "@faker-js/faker";

// 创建模型
export const db = factory({
  user: {
    id: primaryKey(faker.datatype.uuid),
    name: faker.name.findName,
    age: () => faker.datatype.number({ min: 18, max: 120 }),
    city: faker.address.cityName,
    birthday: faker.date.birthdate,
    org: oneOf("organization")
  },
  organiztion: {
    id: primaryKey(faker.datatype.uuid),
    name: faker.company.companyName
  }
});

// 初始化数据
Array.from({ length: 20 }).forEach(() => {
  db.user.create();
});
```

---

# Mock Service Worker

```js{all|2-4|3|6-9|8|11-19|13-17}
const handlers = [
  rest.get("/users", (req, res) => {
    return res(db.user.getAll());
  }),

  rest.post("/users", (req, res) => {
    const body = req.body;
    return res(db.user.create(body));
  }),

  rest.get("/users/:id", (req, res) => {
    const id = req.params.id;
    const user = db.user.findFirst({
      where: {
        id: { equals: id }
      }
    });
    return res(user);
  })
];
```
--- 

# Mock Service Worker

```js{2-12|5-10|14-22|16-20}
const handlers = [
  rest.put("/users/:id", (req, res) => {
    const body = req.body;
    const id = req.params.id;
    const user = db.user.update({
      where: {
        id: { equals: id }
      },
      data: body
    });
    return res(user);
  }),

  rest.delete("/users/:id", (req, res) => {
    const id = req.params.userId;
    const user = db.user.delete({
      where: {
        id: { equals: id }
      }
    });
    return res(user);
  })
];
```

--- 

# Mock Service Worker

```js{all|5-9}
import { response, context } from "msw";

export function successRes(data, ...transformers) {
  return response(
    context.json({
      success: true,
      data: data,
      message: "success"
    }),
    ...transformers
  );
}
```

---
layout: title-two-cols
---

# Mock Service Worker

::left::
**优点**
<v-clicks>

- 可以在 `Network` 面板上看到发出的网络请求
- 可以自定义模型的主键
- 查询数据的方式更灵活
- 支持热更新，支持浏览器调试
- 支持 Browser / Node，支持 REST / GraphQL

</v-clicks>

::right::

**缺点**
<v-clicks>

- 可扩展性带来 API 笨重
- 初始化数据不方便

</v-clicks>

---
layout: cover
class: text-center
background: ""
---

# Demo

---

# 总结

<v-clicks>

- 无缝衔接
  - MSW == JSON-Server Module > Mirage > JSON-Server JSON
- 简单易用
  - JSON-Server JSON > Mirage > MSW > JSON-Server Module
- 综合推荐 MSW
- 但是，三个库各有优缺点，可以吸纳各自的优点

</v-clicks>

--- 

# References

- [Mock Service Worker](https://mswjs.io/)
- [@mswjs/data](https://github.com/mswjs/data)
- [Mirage](https://miragejs.com/)
- [JSON Server](https://github.com/typicode/json-server)
- [Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API)
- [Pretender](https://github.com/pretenderjs/pretender)
- [Express](http://expressjs.com/)
- [Faker](https://fakerjs.dev/)
- [Mock.js](http://mockjs.com/)
- [Chance](https://chancejs.com/)
