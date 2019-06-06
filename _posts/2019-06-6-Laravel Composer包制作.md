---
layout: post
title: 'Laravel Composer包的制作'
date: 2019-06-06
author: 朝夕
cover: ''
tags: composer php Laravel Lumen
---
## 初始化
先新安装Laravel 或者 Lumen项目 (旧的项目也可以，但是需要注意文件的修改)

再项目的根目录下面创建一个文件`packages`(名称随意), 让后在文件夹中创建新包文件

### 在`packages`文件下面创建对应的项目文件夹
> 示例文件结构如下
```
├── chujc // 项目系列名称 或者作者名称
│   └── validation-support // 包名
│       ├── README.md
│       ├── composer.json
│       ├── demo.php
│       └── src
│           ├── Rules
│           │   ├── BackCard.php
│           │   ├── IdCard.php
│           │   └── Password.php
│           ├── Validate.php
│           └── ValidationServiceProvider.php
```
### 在包名称文件下面 生成composer.json
> 在对应的项目包名中执行如下命令 （如果没有安装composer命令环境可以直接复制下面的示例修改）
```bash
composer init
```
根据提示输入对应的内容

```JSON
{
    "name": "chujc/amap-district",  //包名
    "description": "高德地图 中国行政区 导入工具与SDK", //包的简介
    "keywords": ["china", "district", "amap", "laravel"], //搜索关键字
    "license": "MIT", //开源协议
    "authors": [
        //作者
        {
            "name": "john_chu",
            "email": "john1668@qq.com"
        }
    ],
    "require": {
         "php": ">=7.0"  // 依赖, 如果有依赖其他项目的,需要写明版本
    },
    "extra": {
        //laravel5.5以上版本专用的 自动注册 Lumen需要手动注册
        "laravel": {
            "providers": [
                "ChuJC\\AMapDistrict\\DistrictServiceProvider"
            ]
        }
    },
    "autoload": {
        "psr-4": {
            //命名空间的加载路径
            "ChuJC\\AMapDistrict\\": "src/"
        }
    }
}
```
### 添加开发的composer包到项目中
在项目的根目录下面的`composer.json`的文件中添加正在开发的包（注意名称需要与之前的composer名称一致）

添加完成后在命令行执行 `composer update` 项目就会自动加载您定义的扩展包了
```json
{
    ... 
    "repositories": [
        {
            "type": "path", // 固定
            "url": "packages/chujc/amap-district",
 //保持路径与包的规划路径一致既可， composer加载时会去扫码 对应目录下面的composer.json文件           
        },
    ],
    "require": {
        ...
        "chujc/amap-district": "dev-master",
        "chujc/validation-support": "dev-master"
    }
    "require-dev": {
        ...
    },
    ...
}
```

如果遇到使用 composer镜像的情况 可以把 `repositories`做如下修改 ... 表示 镜像配置
```json
{
  "repositories": {
      ...,
      "0": {
          "type": "path",
          "url": "packages/chujc/amap-district"
      },
      "1": {
          "type": "path",
          "url": "packages/chujc/amap-district"
       }
  }
}
```



### 创建ServiceProvider
ServiceProvider需要继承`Illuminate\Support\ServiceProvider`。

ServiceProvider有两个方法`public function register()` 与 `public function boot()` 我们可能需要去实现

具体的就看我们的包应用场景了 `register` 优先于 `boot` 执行，laravel(Lumen)项目中所有的`register`执行后才会依次执行 `boot`方法

`register`注册我们需要后面会用到服务 `boot` 可以执行已经注册的服务, 而不用向在`register`下面担心服务还没有被注册的尴尬.
#### 常用注册方法
- 注册绑定服务
    ```php
    $this->app->singleton('', function($app) {
    })
    ```
示例 注册了command命令 
```php
 $this->app->singleton('command.district',
            function ($app) {
                return new DistrictCommand();
            }
        );
```
- 注册资源发布
```php
// 注册资源发布 第一个参数 键指资源目前的目录，值指发布到对应的目录， 第二个参数可以做标记 可以忽略
$this->publishes([__DIR__.'/amap-district.php' => config_path('district.php')], 'config');
```
```bash
// 执行命令
php artisan vendor:publish --provider="ChuJC\AMapDistrict\DistrictServiceProvider" --tag="config"
// 如果注册时没有第二个参数 则执行时不要带上 --tag
php artisan vendor:publish --provider="ChuJC\AMapDistrict\DistrictServiceProvider"
```
- 添加`command`命令 
  > command命令注册后，还需要添加到执行中才能执行的
```php
$this->commands('command.district');
```

#### 注册ServiceProvider
Laravel 参照上面的composer.json的设置可以实现自动注册

lumen在`bootstrap/app.php` 文件下面
```php
$app->register(ChuJC\Validation\ValidationServiceProvider::class);
```



### 项目提交
[参考](https://chujc.github.io/2019/05/04/composer%E5%88%B6%E4%BD%9C.html)
