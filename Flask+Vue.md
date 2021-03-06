

# Vue+Flask 前后端分离

## 环境配置

### 虚拟环境Pipenv的使用

对于使用Flask、Django等Python框架的项目的开发，虚拟环境的使用非常有必要

具体安装及使用方法见文件Pipenv

### Vue

装Node.js、Vue.js

npm不好用，直接使用淘宝npm镜像cnpm

```shell
# 查看版本
$ npm -v
2.3.0

# 升级 npm
cnpm install npm -g

# 升级或安装 cnpm
npm install cnpm -g

# 最新稳定版
$ cnpm install vue
```

Vue.js 提供一个官方命令行工具，可用于快速搭建大型单页应用

```shell
# 全局安装 @vue/cli
$ cnpm install -g @vue/cli

# 创建一个Vue项目
$ vue create frontend
# 这里需要进行一些配置，默认回车即可

? Vue build? Runtime only
? Use ESLint to lint your code? No
? Setup unit tests with Karma + Mocha? No
? Setup e2e tests with Nightwatch? No
```

Note:

- ESLint代码格式检查实在过于严格，此处不选
- 单元测试的内容不用选

### Flask

创建后端Flask项目前，先创建好虚拟环境

```shell
pipenv install # 创建虚拟环境
pipenv shell # 进入虚拟环境
(venv) pip install Flask # 安装
(venv) flask run # 运行
```

### Ant Design UI

[Ant Design Vue](https://www.antdv.com/docs/vue/introduce-cn/)

```shell
cnpm install ant-design-vue --save # 安装
```
- 在Vue目录中，修改`src/main.js`
- 按需加载，修改配置

### 跨域

安装`flask_cors` 包

```
pip install -U flask-cors
```

使用flask_cors的CORS，代码示例

```python
from flask_cors import CORS
cors = CORS(app, resources={r"/api/*": {"origins": "*"}}
```

## 前后端分离实例

- 为Flask+Vue前后端的项目创建文件夹，如demo
- 创建Vue前端(frontend)

```shell
mkdir demo
cd demo
vue create frontend #具体如何设置见上
```

- 在VS Code中open folder，选择frontend
- 在/frontend的根目录下，创建文件vue.config.js

```js
module.exports = {
    assetsDir: 'static',
    devServer: {
        proxy: 'http://localhost:5000/'
    }
}
```

- 创建虚拟环境+后端Flask
- 在/demo根目录下，打开cmd

```shell
pipenv install
pipenv shell
(venv) pip install Flask
```

- 在/demo根目录下，创建文件app.py

```python
from flask import Flask, render_template, jsonify
from random import *

# 指定了静态和模板文件夹来用前端包指向 /dist 文件夹，在根文件夹中运行 Flask 服务
app = Flask(__name__,
            static_folder = "./frontend/dist/static",
            template_folder = "./frontend/dist")

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def catch_all(path):
    return render_template("index.html")

# 提供api，在后端生成随机数
@app.route('/api/random')
def random_number():
    response = {
        'randomNumber': randint(1, 100)
    }
    return jsonify(response)

if __name__ == "__main__":
    app.run(debug = True)
```

```shell
(venv) flask run # 打开Flask端项目
```

在frontend文件夹中安装

```shell
(venv) npm install --save axios # 安装axios库
```

在Vue项目中，修改Home.vue

```vue
<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/>
    <p>Random number from backend: {{ randomNumber }}</p>
    <button @click="getRandom">New random number</button>
  </div>
</template>

<script>
import axios from 'axios' 
// 导入 axios 库。getrandomfrombackend方法将使用 AXIOS 异步访问 API 并检索结果。最后，方法 getRandom 现在可使用 getRandomFromBackend 函数来获取随机值。
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'

export default {
  name: 'Home',
  components: {
    HelloWorld
  },
  data () {
    return {
      randomNumber: 0
    }
  },
  methods: {
    getRandom () {
      // this.randomNumber = this.getRandomInt(1, 100)
      this.randomNumber = this.getRandomFromBackend()
    },
    getRandomFromBackend () {
      const path = `http://localhost:5000/api/random`
      axios.get(path)
      .then(response => {
        this.randomNumber = response.data.randomNumber
      })
      .catch(error => {
        console.log(error)
      })
    }
  },
  created () {
    this.getRandom()
  }
}
</script>
```

Note:

- 在VS Code中的NPM SCRIPTS，build并serve
- 刷新 localhost:8080，会发现没有随机值是正常现象。因为Flask服务器API默认关闭到其他Web服务器
- 在管理Flask的cmd里，ctrl+c重启项目，再打开，便当可得到想要的生成随机数效果