---
title: "meteor-加载静态文件"
date: 2020-02-11T14:07:25+08:00
---
meteor 加载静态文件

<!--more-->

meteor 加载外部资源文件

在很多时候开发的时候我们需要导入一些外部的数据到程序当中，这些数据可能存放在csv,XML.json之类的文件当中，我们就将其导入到程序当中，使其能够供全局进行访问。

private/json/city.json

     [
      {"code":"010","city":"北京", "pinyin": "beijing"},
      {"code":"021","city":"上海", "pinyin":"shanghai"},
      {"code":"0571","city":"杭州", "pinyin":"hangzhou","province": "浙江"},
      {"code":"0574","city":"宁波", "pinyin":"ningbo","province": "浙江"},
      {"code":"0573","city":"嘉兴", "pinyin":"jiaxing","province": "浙江"},
      {"code":"0575","city":"绍兴", "pinyin":"shaoxing","province": "浙江"},
      {"code":"0577","city":"温州", "pinyin":"wenzhou","province": "浙江"},
      {"code":"0580","city":"舟山", "pinyin":"zhoushan","province": "浙江"},
      {"code":"0572","city":"湖州", "pinyin":"huzhou","province": "浙江"},
      {"code":"0579","city":"金华", "pinyin":"jinhua","province": "浙江"},
      {"code":"0578","city":"丽水", "pinyin":"lishui","province": "浙江"},
      {"code":"0576","city":"台州", "pinyin":"taizhou","province": "浙江"},
      {"code":"0570","city":"衢州", "pinyin":"quzhou","province": "浙江"},
      ]

特殊注意是private文件夹的数据是不提供给客户端

上面的是包含城市名称以及区号的json数据，我们将其保存在文件当中，这里我们可以通过Assets API来访问以及处理相应的数据。

server/main.js

    import { Meteor } from 'meteor/meteor';
    
    Meteor.startup(function(){
      var cityList=JSON.parse(Assets.getText('json/city.json'))
      _.each(cityList, function(city) {
        console.log(city);
      });
    })

上面的代码会将所有的数据进行输出，但是有的时候这些数据可以会被其他的项目需要，这里我们需要通过添加一个package来解决，同时我们只可以将这个包上传到atmospherejs

我们需要将之前的json文件移到cityjson包，最终的目录文件如下所示

    └── cityjson
        ├── README.md
        ├── city.json
        ├── cityjson-tests.js
        ├── cityjson.js
        └── package.js

package.js

    Package.describe({
      name: 'wenchangshou:cityjson',
      version: '0.0.1',
      // Brief, one-line summary of the package.
      summary: '获取国内的所有的城市名称以及区号',
      // URL to the Git repository containing the source code for this package.
      git: '',
      // By default, Meteor will default to using README.md for documentation.
      // To avoid submitting documentation, set this field to null.
      documentation: 'README.md'
    });
    
    Package.onUse(function (api) {
      //  api.versionsFrom('1.4.2.7');
      api.versionsFrom('1.0')
      api.export('cityList', 'server');
      api.addAssets('city.json', 'server')
      api.addFiles('cityjson.js', 'server')
    });
    
    
    Package.onTest(function(api) {
      api.use('ecmascript');
      api.use('tinytest');
      api.use('wenchangshou:cityjson');
      api.mainModule('cityjson-tests.js');
    });
    

cityjson.js

    var cityCollection=JSON.parse(Assets.getText('city.json'));
    cityList={city:cityCollection}

- Assets.getText 可以从当前的包内进行调用 ，通过addFiles进行数据的暴露
- 之后我们需要读取以及解析数据，city里面包含了所有的城市信息，并且我们通过cityList这个全局变量进行数据的暴露
- 我们通过 api.addAssets将静态的文件导入到包中，通过Assets.getText进行访问

meteor 包上传

上面的操作我们已经创建了一个能够使用的包了，这时候我们需要将这个包上传，方便将来的使用

meteor publish --create 

上传成功之后我们可以直接访问cityjson来查看我们上传的包，如下图所示



上传成功之后我们将pckages/cityjson目录删除

rm -rf packages/cityjson

我们在主目录当中添加我们上传的包,其中wenchangshou是我的用户名

meteor add wenchangshou:cityjson
