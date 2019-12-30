查看模板中使用

``` javascript
 {
     %
     remote key = "recommend", api = "ads/getOne", query = '{"name":"recommend"}' %
 }
```

第一步：全局引入

``` javascript
// app/bootstrap/index.js 

require('module-alias/register')
require('./global');
require('./tags'); // 这个就是nunjucks自定义一个请求的标签
```

第二步：实现tag

```javascript 
// app/bootstrap/tags.js

'use strict'; 
const _ = require('lodash'); 

global.remote = function (appCtx) {

    this.tags = ['remote'];
    this.parse = function (parser, nodes, lexer) {
        var tok = parser.nextToken();
        var args = parser.parseSignature(null, true);
        parser.advanceAfterBlockEnd(tok.value);
        return new nodes.CallExtensionAsync(this, 'run', args);

    };

    this.run = async (context, args, callback) => {
        const key = _.isEmpty(args.key) ? false : args.key;
        // console.log('---key--', key);
        try {
            const api = _.isEmpty(args.api) ? false : args.api;
            const queryObj = _.isEmpty(args.query) ? false : JSON.parse(args.query);
            const pageSize = _.isEmpty(args.pageSize) ? false : args.pageSize;
            const isPaging = _.isEmpty(args.isPaging) ? '1' : args.isPaging;

            let apiData = [];

            if (!key || !api) {
                throw new Error(context.ctx.__('validate_error_params'));
            }

            let payload = {};

            if (pageSize) {
                payload.pageSize = pageSize;
            }

            if (isPaging) {
                payload.isPaging = isPaging;
            }

            if (queryObj) {
                _.assign(payload, queryObj);
            }

            apiData = await appCtx.helper.reqJsonData(api, payload);
            // console.log(payload, '--apiData--', apiData);
            context.ctx[key] = apiData;
            return callback(null, '');
        } catch (error) {
            context.ctx[key] = [];
            return callback(null, '');
        }

    };

}

``` 

第三步：注册标签

```javascript
// 根目录app.js
const path = require('path');

class AppBootHook {
    constructor(app) {
        this.app = app;
    }

    configWillLoad() {
        // 加载全局文件
        this.app.loader.loadFile(path.join(this.app.config.baseDir, 'app/bootstrap/index.js'));
        // 获取ctx上下文
        const ctx = this.app.createAnonymousContext();
        // 添加扩展（注册这个标签）
        this.app.nunjucks.addExtension('remote', new remote(ctx));
    }
}
module.exports = AppBootHook;

```

