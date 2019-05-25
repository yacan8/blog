---
title: Vue+Express+Mysql 全栈初体验
date: 2019-05-25 17:37:56
tags:
- 全栈
- 前端
- javascript
- vue.js
---

## 前言

曾几何时，你有没有想过一个前端工程师的未来是什么样的？这个时候你是不是会想到了一个词”前端架构师“，那么一个合格的前端架构只会前端OK吗？那当然不行，你必须具备全栈的能力，这样才能扩大个人的形象力，才能升职加薪，才能迎娶白富美，才能走向人生巅峰...

最近我在写一些后端的项目，发现重复工作太多，尤其是框架部分，然后这就抽空整理了前后端的架子，主要是用的Vue，Express，数据存储用的Mysql，当然如果有其他需要，也可以直接切换到sqlite、postgres或者mssql。

先献上[项目github](https://github.com/yacan8/express-vue-web-slush)

## 项目

项目以todolist为🌰，简单的实现了前后端的CURD。

### 后端技术栈

*   [框架 Express](http://expressjs.com/)
*   [热更新 nodemon](https://github.com/remy/nodemon)
*   [依赖注入 awilix](https://github.com/jeffijoe/awilix)
*   [数据持久化 sequelize](https://github.com/sequelize/sequelize)
*   [部署 pm2](https://pm2.io/)

### 前端技术栈

*   [vue-router](https://router.vuejs.org)
*   [vuex](https://vuex.vuejs.org)
*   [axios](https://github.com/axios/axios)
*   [vue-class-component](https://github.com/vuejs/vue-class-component)
*   [vue-property-decorator](https://github.com/kaorun343/vue-property-decorator#readme)
*   [vuex-class](https://github.com/ktsn/vuex-class)

### 项目结构

先看项目架构，client为前端结构，server为后端结构

```
|-- express-vue-web-slush
    |-- client
    |   |-- http.js   // axios 请求封装
    |   |-- router.js  // vue-router
    |   |-- assets  // 静态资源
    |   |-- components  // 公用组件
    |   |-- store  // store
    |   |-- styles // 样式
    |   |-- views // 视图
    |-- server
        |-- api    // controller api文件
        |-- container  // ioc 容器
        |-- daos  // dao层
        |-- initialize  // 项目初始化文件
        |-- middleware  // 中间件
        |-- models  // model层
        |-- services // service层
```

### 代码介绍

前端代码就不多说，一眼就能看出是vue-cli生成的结构，不一样的地方就是前端编写的代码是以Vue Class的形式编写的，具体细节请见[从react转职到vue开发的项目准备](https://github.com/yacan8/blog/blob/master/posts/%E4%BB%8Ereact%E8%BD%AC%E8%81%8C%E5%88%B0vue%E5%BC%80%E5%8F%91%E7%9A%84%E9%A1%B9%E7%9B%AE%E5%87%86%E5%A4%87.md)

然后这里主要描述一下后端代码。

#### 热更新

开发环境必需品，我们使用的是nodemon，在项目根目录添加`nodemon.json`：

```json
{
  "ignore": [
    ".git",
    "node_modules/**/node_modules",
    "src/client"
  ]
}
```

`ignore`忽略 node_modules 和 前端代码文件夹src/client 的js文件变更，ignore以外的js文件变更nodemon.json会重启node项目。

这里为了方便，我写了一个脚本，同时启动前后端项目，如下：

```js
import * as childProcess from 'child_process';

function run() {
  const client = childProcess.spawn('vue-cli-service', ['serve']);
  client.stdout.on('data', x => process.stdout.write(x));
  client.stderr.on('data', x => process.stderr.write(x));

  const server = childProcess.spawn('nodemon', ['--exec', 'npm run babel-server'], {
    env: Object.assign({
      NODE_ENV: 'development'
    }, process.env),
    silent: false
  });
  server.stdout.on('data', x => process.stdout.write(x));
  server.stderr.on('data', x => process.stderr.write(x));

  process.on('exit', () => {
    server.kill('SIGTERM');
    client.kill('SIGTERM');
  });
}
run();
```

前端用vue-cli的`vue-cli-service`命令启动。

后端用`nodemon`执行`babel-node命令启动`。

然后这前后端项目由node子进程启动，然后我们在package.json里添加script。

```json
{
    "scripts": {
        "dev-env": "cross-env NODE_ENV=development",
        "babel-server": "npm run dev-env && babel-node --config-file ./server.babel.config.js -- ./src/server/main.js",
        "dev": "babel-node --config-file ./server.babel.config.js -- ./src/dev.js",
    }
}
```

`server.babel.config.js`为后端的bable编译配置。

#### 项目配置

所谓的项目配置呢，说的就是与业务没有关系的系统配置，比如你的日志监控配置、数据库信息配置等等

首先，在项目里面新建配置文件，`config.properties`，比如我这里使用的是Mysql，内容如下：

```
[mysql]
host=127.0.0.1
port=3306
user=root
password=root
database=test
```

在项目启动之前，我们使用[properties](https://www.npmjs.com/package/properties)对其进行解析，在我们的`server/initialize`新建`properties.js`，对配置文件进行解析：

```js
import properties from 'properties';
import path from 'path';

const propertiesPath = path.resolve(process.cwd(), 'config.properties');

export default function load() {
  return new Promise((resolve, reject) => {
    properties.parse(propertiesPath, { path: true, sections: true }, (err, obj) => {
      if (err) {
        reject(err);
        return;
      }
      resolve(obj);
    });
  }).catch(e => {
    console.error(e);
    return {};
  });
}
```

然后在项目启动之前，初始化mysql，在`server/initialize`文件夹新建文件`index.js`

```js
import loadProperties from './properties';
import { initSequelize } from './sequelize';
import container from '../container';
import * as awilix from 'awilix';
import { installModel } from '../models';

export default async function initialize() {
  const config = await loadProperties();
  const { mysql } = config;
  const sequelize = initSequelize(mysql);
  installModel(sequelize);
  container.register({
    globalConfig: awilix.asValue(config),
    sequelize: awilix.asValue(sequelize)
  });
}
```

这里我们数据持久化用的sequelize，依赖注入用的awilix，我们下文描述。

初始化所有配置后，我们在项目启动之前执行initialize，如下：

```js
import express from 'express';
import initialize from './initialize';
import fs from 'fs';

const app = express();

export default async function run() {
  await initialize(app);

  app.get('*', (req, res) => {
    const html = fs.readFileSync(path.resolve(__dirname, '../client', 'index.html'), 'utf-8');
    res.send(html);
  });

  app.listen(9001, err => {
    if (err) {
      console.error(err);
      return;
    }
    console.log('Listening at http://localhost:9001');
  });
}

run();
```

#### 数据持久化

作为前端，对数据持久化这个词没什么概念，这里简单介绍一下，首先数据分为两种状态，一种是瞬时状态，一种是持久状态，而瞬时状态的数据一般是存在内存中，还没有永久保存的数据，一旦我们服务器挂了，那么这些数据将会丢失，而持久状态的数据呢，就是已经落到硬盘上面的数据，比如mysql、mongodb的数据，是保存在硬盘里的，就算服务器挂了，我们重启服务，还是可以获取到数据的，所以数据持久化的作用就是将我们的内存中的数据，保存在mysql或者其他数据库中。

我们数据持久化是用的[sequelize](https://github.com/sequelize/sequelize)，它可以帮我们对接mysql，让我们快速的对数据进行CURD。

下面我们在`server/initialize`文件夹新建`sequelize.js`，方便我们在项目初始化的时候连接：

```js
import Sequelize from 'sequelize';

let sequelize;

const defaultPreset = {
  host: 'localhost',
  dialect: 'mysql',
  operatorsAliases: false,
  port: 3306,
  pool: {
    max: 10,
    min: 0,
    acquire: 30000,
    idle: 10000
  }
};

export function initSequelize(config) {
  const { host, database, password, port, user } = config;
  sequelize = new Sequelize(database, user, password, Object.assign({}, defaultPreset, {
    host,
    port
  }));
  return sequelize;
};

export default sequelize;
```

initSequelize的入参config，来源于我们的`config.properties`，在项目启动之前执行连接。

然后，我们需要对应数据库的每个表建立我们的Model，以todolist为例，在`service/models`，新建文件`ItemModel.js`：

```js
export default function(sequelize, DataTypes) {
    const Item = sequelize.define('Item', {
        recordId: {
            type: DataTypes.INTEGER,
            field: 'record_id',
            primaryKey: true
        },
        name: {
            type: DataTypes.STRING,
            field: 'name'
        },
        state: {
            type: DataTypes.INTEGER,
            field: 'state'
        }
    }, {
        tableName: 'item',
        timestamps: false
    });
    return Item;
}
```

然后在`service/models`，新建`index.js`，用来导入models文件夹下的所有model:

```js
import fs from 'fs';
import path from 'path';
import Sequelize from 'sequelize';

const db = {};

export function installModel(sequelize) {
  fs.readdirSync(__dirname)
    .filter(file => (file.indexOf('.') !== 0 && file.slice(-3) === '.js' && file !== 'index.js'))
    .forEach((file) => {
      const model = sequelize.import(path.join(__dirname, file));
      db[model.name] = model;
    });
  Object.keys(db).forEach((modelName) => {
    if (db[modelName].associate) {
      db[modelName].associate(db);
    }
  });
  db.sequelize = sequelize;
  db.Sequelize = Sequelize;
}

export default db;
```

这个`installModel`也是在我们项目初始化的时候执行的。

model初始化完了之后，我们就可以定义我们的Dao层，使用model了。

#### 依赖注入

依赖注入（DI）是反转控制（IOC）的最常用的方式。最早听说这个概念的相信大多数都是来源于Spring，反转控制最大的作用的帮我们创建我们所需要是实例，而不需要我们手动创建，而且实例的创建的依赖我们也不需要关心，全都由IOC帮我们管理，大大的降低了我们代码之间的耦合性。

这里用的依赖注入是[awilix](https://github.com/jeffijoe/awilix)，首先我们创建容器，在`server/container`，下新建`index.js`：

```js
import * as awilix from 'awilix';

const container = awilix.createContainer({
  injectionMode: awilix.InjectionMode.PROXY
});

export default container;
```

然后在我们项目初始化的时候，用[awilix-express](https://www.npmjs.com/package/awilix-express)初始化我们后端的router，如下：

```js
import { loadControllers, scopePerRequest } from 'awilix-express';
import { Lifetime } from 'awilix';

const app = express();

app.use(scopePerRequest(container));

app.use('/api', loadControllers('api/*.js', {
  cwd: __dirname,
  lifetime: Lifetime.SINGLETON
}));
```

然后，我们可以在`server/api`下新建我们的controller，这里新建一个`TodoApi.js`：

```js
import { route, GET, POST } from 'awilix-express';

@route('/todo')
export default class TodoAPI {

  constructor({ todoService }) {
    this.todoService = todoService;
  }

  @route('/getTodolist')
  @GET()
  async getTodolist(req, res) {
    const [err, todolist] = await this.todoService.getList();
    if (err) {
      res.failPrint('服务端异常');
      return;
    }
    res.successPrint('查询成功', todolist);
  }

  //  ...
}
```

这里可以看到构造函数的入参注入了Service层的`todoService`实例，然后可以直接使用。

然后，我们要搞定我们的Service层和Dao层，这也是在项目初始化的时候，告诉IOC我们所有Service和Dao文件：

```js
import container from './container';
import { asClass } from 'awilix';

// 依赖注入配置service层和dao层
container.loadModules(['services/*.js', 'daos/*.js'], {
  formatName: 'camelCase',
  register: asClass,
  cwd: path.resolve(__dirname)
});
```

然后我们可以在services和daos文件夹下肆无忌惮的新建service文件和dao文件了，这里我们新建一个`TodoService.js`：

```js

export default class TodoService {
  constructor({ itemDao }) {
    this.itemDao = itemDao;
  }

  async getList() {
    try {
      const list = await this.itemDao.getList();
      return [null, list];
    } catch (e) {
      console.error(e);
      return [new Error('服务端异常'), null];
    }
  }

  // ...
}
```

然后，新建一个Dao，`ItemDao.js`，用来对接ItemModel，也就是mysql的Item表：

```js
import BaseDao from './base';

export default class ItemDao extends BaseDao {
    
    modelName = 'Item';

    constructor(modules) {
      super(modules);
    }

    async getList() {
      return await this.findAll();
    }
}
```

然后搞一个BaseDao，封装一些数据库的常用操作，代码太长，就不贴了，详情见[代码库](https://github.com/yacan8/express-vue-web-slush/blob/master/src/server/daos/base.js)。

#### 关于事务

所谓事务呢，简单的比较好理解，比如我们执行了两条SQL，用来新增两条数据，当第一条执行成功了，第二条没执行成功，这个时候我们执行事务的回滚，那么第一条成功的记录也将会被取消。

然后呢，我们这里为了也满足事务，我们可以按需使用中间件，为请求注入事务，然后所以在这个请求下执行的增删改的SQL，都使用这个事务，如下中间件：

```js
import { asValue } from 'awilix';

export default function () {
  return function (req, res, next) {
    const sequelize = container.resolve('sequelize');
    sequelize.transaction({  // 开启事务
      autocommit: false
    }).then(t => {
      req.container = req.container.createScope(); // 为当前请求新建一个IOC容器作用域
      req.transaction = t;
      req.container.register({  // 为IOC注入一个事务transaction
        transaction: asValue(t)
      });
      next();
    });
  }
}
```

然后当我们需要提交事务的时候，我们可以使用IOC注入transaction，例如，我们在TodoService.js中使用事务

```js

export default class TodoService {
  constructor({ itemDao, transaction }) {
    this.itemDao = itemDao;
    this.transaction = transaction;
  }

  async addItem(item) {
    // TODO: 添加item数据
    const success = await this.itemDao.addItem(item);
    if (success) {
      this.transaction.commit(); // 执行事务提交
    } else {
      this.transaction.rollback(); // 执行事务回滚
    }
  }

  // ...
}
```

#### 其他

当我们需要在Service层或者Dao层使用到当前的请求对象怎么办呢，这个时候我们需要在IOC中为每一条请求注入request和response，如下中间件：

```js
import { asValue } from 'awilix';

export function baseMiddleware(app) {
  return (req, res, next) => {
    res.successPrint = (message, data) => res.json({ success: true, message, data });

    res.failPrint = (message, data) => res.json({ success: false, message, data });
    req.app = app;

    // 注入request、response
    req.container = req.container.createScope();
    req.container.register({
      request: asValue(req),
      response: asValue(res)
    });
    next();
  }
}
```

然后在项目初始化的时候，使用该中间件：

```js
import express from 'express';

const app = express();
app.use(baseMiddleware(app));
```

#### 关于部署

使用pm2，简单实现部署，在项目根目录新建`pm2.json`

```json
{
  "apps": [
    {
      "name": "vue-express",  // 实例名
      "script": "./dist/server/main.js",  // 启动文件
      "log_date_format": "YYYY-MM-DD HH:mm Z",  // 日志日期文件夹格式
      "output": "./log/out.log",  // 其他日志
      "error": "./log/error.log", // error日志
      "instances": "max",  // 启动Node实例数
      "watch": false, // 关闭文件监听重启
      "merge_logs": true,
      "env": {
        "NODE_ENV": "production"
      }
    }
  ]
}
```

这个时候，我们需要把客户端和服务端编译到dist目录，然后将服务端的静态资源目录指向客户端目录，如下：

```js
app.use(express.static(path.resolve(__dirname, '../client')));
```

添加vue-cli的配置文件`vue.config.js`:

```js
const path = require('path');
const clientPath = path.resolve(process.cwd(), './src/client');
module.exports = {
  configureWebpack: {
    entry: [
      path.resolve(clientPath, 'main.js')
    ],
    resolve: {
      alias: {
        '@': clientPath
      }
    },
    devServer: {
      proxy: {
        '/api': { // 开发环境将API前缀配置到后端端口
          target: 'http://localhost:9001'
        }
      }
    }
  },
  outputDir: './dist/client/'
};
```

在package.json中添加如下script：

```json
{
  "script": {
    "clean": "rimraf dist",
    "pro-env": "cross-env NODE_ENV=production",
    "build:client": "vue-cli-service build",
    "build:server": "babel --config-file ./server.babel.config.js src/server --out-dir dist/server/",
    "build": "npm run clean && npm run build:client && npm run build:server",
    "start": "pm2 start pm2.json",
    "stop": "pm2 delete pm2.json"
  }
}
```

执行build命令，清理dist目录，同时编译前后端代码到dist目录下，然后`npm run start`，pm2启动`dist/server/main.js`;

到此为止，部署完成。

### 结束

发现自己挂羊头卖狗肉，竟然全在写后端。。。好吧，我承认我本来就是想写后端的，但是我还是觉得作为一个前端工程师，Nodejs应该是在这条路上走下去的必备技能，加油~。

[项目github](https://github.com/yacan8/express-vue-web-slush)