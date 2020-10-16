# 欢迎访问 baidu-tongji-with-multiple-channel 官网 👋

[![npm:version](https://img.shields.io/npm/v/gitbook-plugin-baidu-tongji-with-multiple-channel.svg)](https://www.npmjs.com/package/gitbook-plugin-baidu-tongji-with-multiple-channel)
[![npm:download](https://img.shields.io/npm/dt/gitbook-plugin-baidu-tongji-with-multiple-channel.svg)](https://www.npmjs.com/package/gitbook-plugin-baidu-tongji-with-multiple-channel)
[![npm:prerequisite](https://img.shields.io/badge/gitbook-*-blue.svg)](https://www.npmjs.com/package/gitbook-plugin-baidu-tongji-with-multiple-channel)
[![github:documentation](https://img.shields.io/badge/documentation-yes-brightgreen.svg)](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel#readme)
[![github:maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/graphs/commit-activity)
[![npm:license](https://img.shields.io/npm/l/gitbook-plugin-baidu-tongji-with-multiple-channel.svg)](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/blob/master/LICENSE)
[![github:snodreams1006](https://img.shields.io/badge/github-snowdreams1006-brightgreen.svg)](https://github.com/snowdreams1006)
[![website:snodreams1006.tech](https://img.shields.io/badge/website-snowdreams1006.tech-brightgreen.svg)](https://snowdreams1006.tech/)
[![微信公众号:雪之梦技术驿站-brightgreen.svg](https://img.shields.io/badge/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7-%E9%9B%AA%E4%B9%8B%E6%A2%A6%E6%8A%80%E6%9C%AF%E9%A9%BF%E7%AB%99-brightgreen.svg)](https://snowdreams1006.github.io/snowdreams1006-wechat-public.jpeg)

> Gitbook 百度统计插件,支持多渠道独立统计,一份源码多处部署独立统计.

### 🏠 [主页](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel)

- Github : [https://snowdreams1006.github.io/gitbook-plugin-baidu-tongji-with-multiple-channel/](https://snowdreams1006.github.io/gitbook-plugin-baidu-tongji-with-multiple-channel/)
- GitLab: [https://snowdreams1006.gitlab.io/gitbook-plugin-baidu-tongji-with-multiple-channel/](https://snowdreams1006.gitlab.io/gitbook-plugin-baidu-tongji-with-multiple-channel/)
- Gitee : [https://snowdreams1006.gitee.io/gitbook-plugin-baidu-tongji-with-multiple-channel/](https://snowdreams1006.gitee.io/gitbook-plugin-baidu-tongji-with-multiple-channel/)

## 预览

**用法**

````json
{
    "plugins": [
        "baidu-tongji-with-multiple-channel"
    ],
    "pluginsConfig": {
        "baidu-tongji-with-multiple-channel": {
            "token": "5273b2f99de3bc190886abb53f62267e"
        }
    }
}
````

**效果**

```js
<script src="https://hm.baidu.com/hm.js?5273b2f99de3bc190886abb53f62267e"></script>
```

## 用法

### 步骤＃1-更新 `book.json` 文件

在您的 `gitbook` 的 `book.json` 文件中,将 `baidu-tongji-with-multiple-channel` 添加到 `plugins` 列表中.

这是最简单的示例: 

```json
{
    "plugins": ["baidu-tongji-with-multiple-channel"]
}
```

此外,受支持的配置选项如下:

```json
"gitbook": {
    "properties": {
        "token": {
          "description": "百度统计token",
          "required": false,
          "type": "string"
        },
        "url": {
          "description": "百度统计地址",
          "default":"https://hm.baidu.com/hm.js",
          "required": false,
          "type": "string"
        },
        "multipleChannelConfig": {
            "description": "百度统计多渠道配置",
            "required": false,
            "type": "object"
        }
    }
}
```

### 步骤＃2- 运行 `gitbook` 命令

- 运行 `gitbook install` .它将自动为您的 `gitbook` 安装 `baidu-tongji-with-multiple-channel` 插件.

> 该步骤仅需要允许一次即可.

```bash
gitbook install
```

或者您可以运行 `npm install gitbook-plugin-baidu-tongji-with-multiple-channel` 命令本地安装 `gitbook-plugin-baidu-tongji-with-multiple-channel` 插件.

```bash
npm install gitbook-plugin-baidu-tongji-with-multiple-channel
```

- 像往常一样构建您的书（ `gitbook build` ）或服务（ `gitbook serve` ）.

```bash
gitbook serve
```

### 步骤＃3- 验证是否注入百度统计代码

上传网站后访问网站首页检查是否自动注入百度统计脚本文件,也在百度统计后台检测是否配置成功.

```js
<script src="https://hm.baidu.com/hm.js?5273b2f99de3bc190886abb53f62267e"></script>
```

## 申请

### 步骤＃1- 注册百度统计网站

登录[百度统计官网](https://tongji.baidu.com/),注册或者登录百度统计后台,点击**新增网站**并填写网站相关信息.

|名称|示例|备注|
|:-:|:-:|:-:|
|网站域名|`snowdreams1006.github.io`|支持主域名,二级域名,子目录和wap站域名|
|网站首页|`https://snowdreams1006.github.io/`|必选|
|网站名称|雪之梦技术驿站|可选|
|行业类别|博客-教育|可选|

### 步骤＃2 - 获取百度统计代码

**新增网站**成功后,点击**代码获取**,复制统计代码脚本如下:

```js
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?5273b2f99de3bc190886abb53f62267e";
  var s = document.getElementsByTagName("script")[0]; 
  s.parentNode.insertBefore(hm, s);
})();
```

|名称|示例|备注|
|:-:|:-:|:-:|
|源地址|`https://hm.baidu.com/hm.js?5273b2f99de3bc190886abb53f62267e`|统计脚本地址|
|url|`https://hm.baidu.com/hm.js`|每个网站的值相同|
|token|`5273b2f99de3bc190886abb53f62267e`|每个网站的值不同|

### 步骤＃3- 填写百度统计配置

复制刚刚申请的 `token` 以及 `url` 填写到 `book.json` 配置文件中,按照实际情况填写百度统计配置信息.

- 单渠道配置仅仅需要填写 `token` 的值即可,其余选项可选.

```json
"baidu-tongji-with-multiple-channel": {
  "token": "5273b2f99de3bc190886abb53f62267e"
}
```

- 多渠道除了需要填写**网站域名**对应的 `token` 的值,还需要填写百度统计 `url`.

```json
"baidu-tongji-with-multiple-channel": {
  "url": "https://hm.baidu.com/hm.js",
  "multipleChannelConfig": {
      "snowdreams1006.tech":{
          "token": "468a97b20b79bc025d27afbee73d2f39"
      },
      "blog.snowdreams1006.cn":{
          "token": "606f5db455f771889dcfd16bb2bd313b"
      },
      "snowdreams1006.github.io":{
          "token": "5273b2f99de3bc190886abb53f62267e"
      },
      "snowdreams1006.gitee.io":{
          "token": "60ec676f32be4579eb53a430e153f677"
      },
      "snowdreams1006.gitlab.io":{
          "token": "187a0e5601c4e4987cdb5b8df09c9b21"
      },
      "snowdreams1006.gitbook.io":{
          "token": "6e2ee1b7bea146b2d6d9382bbf803083"
      }
  }
}
```

## 示例

- 官方文档配置文件

> [https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/blob/master/docs/book.json](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/blob/master/docs/book.json)

```json
{
    "plugins": ["baidu-tongji-with-multiple-channel"],
    "pluginsConfig": {
        "baidu-tongji-with-multiple-channel": {
            "method": "baidu-tongji-with-multiple-channelJson"
        }
    }
}
```

- 官方示例配置文件

> [https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/blob/master/example/book.json](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/blob/master/example/book.json)

```json
{
    "plugins": ["baidu-tongji-with-multiple-channel"],
    "pluginsConfig": {
        "baidu-tongji-with-multiple-channel": {
            "method": "baidu-tongji-with-multiple-channelJson"
        }
    }
}
```

**注意** :如果您的书还没有创建,以上代码段可以用作完整的 `book.json` 文件.

## 致谢

- baidu + cnzz + google analytics for gitbook plugin . : [https://github.com/xzghua/gitbook-plugin-statistics](https://github.com/xzghua/gitbook-plugin-statistics)
- GitBook 插件：百度统计 : [https://github.com/huisman6/gitbook-plugin-baidu-tongji](https://github.com/huisman6/gitbook-plugin-baidu-tongji)

## 作者

👤 **snowdreams1006**

- 网站 : [snowdreams1006.tech](https://snowdreams1006.tech/)
- GitHub :  [@snowdreams1006](https://github.com/snowdreams1006)
- 电子邮件 : [snowdreams1006@163.com](mailto:snowdreams1006@163.com)

## 贡献

欢迎贡献问题和功能需求,随时检查[问题页面](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/issues) .

## 支持

如果这个项目对您有帮助,请给个[星星](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel) !

## 版权

版权所有©2019 [snowdreams1006](https://github.com/snowdreams1006).

该项目是[MIT](https://github.com/snowdreams1006/gitbook-plugin-baidu-tongji-with-multiple-channel/blob/master/LICENSE)许可的.