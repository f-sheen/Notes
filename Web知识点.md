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

## 2、