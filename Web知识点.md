## 1、Vue配置生产环境和测试环境

- 在根目录创建文件

  .env 无论什么环境都会加载
  .env.production 生产环境加载
  .env.development 测试开发环境加载

- 配置文件

  .env文件：

  ```javascript
  VUE_APP_NAME='vue测试名称'
  ```

  .env.development文件：

  ```javascript
  NODE_ENV = development  
  VUE_APP_URL = 'developmentURL' //自定义变量  必须要以VUE_APP_开头定义
  ```

  .env.production 文件：

  ```javascript
  NODE_ENV = production    
  VUE_APP_URL = 'productionURL' //自定义变量  必须要以VUE_APP_开头定义
  ```

  

- 配置package.json

  ```json
  {
    "scripts": {
      "serve": "vue-cli-service serve",
      //打包测试开发版本  
      //--mode 后面需要对用文件的名字.env.development  重点是要和.env.后面的名字对应起来
      "build-development": "vue-cli-service build --mode development",
      //打包生产版本
      //--mode 后面需要对用文件的名字.env.production 重点是要和.env.后面的名字对应起来
      "build-production": "vue-cli-service build --mode production",
      "lint": "vue-cli-service lint"
    },
    ...//json文件太长 后面的没用复制过来  主要看上面的scripts
  }
  ```

  最后通过npm run build-production，生成dist文件夹，通过全局安装http-server这个npm包，进入dist文件夹打开cmd，执行http-server命令

<<<<<<< HEAD
## 2、空值合并运算符??和逻辑或||

1. 空值合并运算符??
   - 触发条件：仅当左侧为 null 或 undefined 时返回右侧值
2. 逻辑或||
   - 触发条件：左侧为 假值 时返回右侧值，假值有false、0、""、NaN、null、undefined

3、
=======
## 2、单向数据流原则

​	父组件通过props向子组件传递数据，子组件通过事件向父组件通信。
​	数据从父组件流向子组件，子组件不能直接修改父组件传递过来的props，而是通过触发事件让父组件来修改数据。

## 3、URL编码问题

%20 是空格，%3A 是冒号 :
当前端传递2025-12-24%2012%3A02%3A03时，后端会自动解码为2025-12-24 12:02:03

## 4、响应式

- ref：

  - 创建一个响应式的引用对象（wrapper）
  - 返回的是RefImpl 对象（带 .value 属性）
  - 主要用于基本类型（string, number, boolean）或单个值
  - 在 script 中访问/修改必须加 .value
  - 在 template 中自动解包（不用 .value）
  -  如果把 ref 放进 reactive 对象里，它也会被自动解包！

- reactive：

  - 创建一个响应式的对象/数组
  - 返回的是Proxy对象
  - 适用于复杂数据结构（object, array）
  - 直接访问属性，无需 .value
  - 不能直接替换整个对象（会丢失响应性）
  - 不能用于基本类型

- toraw

  - 获取响应式对象的原始（非代理）版本
  - reactive() 返回的是 Proxy 对象，某些库（如 Lodash、JSON.stringify）可能不兼容 Proxy
  - 不需要对toRaw使用
  - 对 raw 的修改不会触发响应式更新

- ...扩展运算符

  - 创建普通对象副本，剥离响应式，但浅拷贝
  - 浅拷贝复制的是引用，不是值！

  
>>>>>>> b015080a6f18e4cda7d3b0e911b61c44816b8349
