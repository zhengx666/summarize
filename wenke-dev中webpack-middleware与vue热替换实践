###wenke-dev中webpack-middleware与vue热替换实践
模块热替换是webpack提供的最有用的功能之一，对于vue来说，由vue-cli生成的脚手架工具展示了如何使用webpack-dev-server实现热替换，但是对于前后端没有分离的业务或者有自己定制的打包工具，基于webpack-middleware定制热加载方案更加方便。

对于一般的热替换js代码，业务代码需要侵入一部分代码以实现模块的热替换。如

```
  import _ from 'lodash';
  import printMe from './print.js';

  function component() {
    var element = document.createElement('div');
    var btn = document.createElement('button');

    element.innerHTML = _.join(['Hello', 'webpack'], ' ');

    btn.innerHTML = 'Click me and check the console!';
    btn.onclick = printMe;

    element.appendChild(btn);

    return element;
  }

  document.body.appendChild(component());
+
+ if (module.hot) {
+   module.hot.accept('./print.js', function() {
+     console.log('Accepting the updated printMe module!');
+     printMe();
+   })
+ }
```

vue-loader集成了vue-hot-reload-api,这部分工作可以省去，实现一种开箱即用的效果。vue的dev server只需要实现两个部分，在server端提供热加载服务，将编译好的代码写入内存，等待请求。在client端建立与devserver端的连接，监听变化，更新模块。

首先是把webpack-hot-middleware/client注入js入口文件，如

```
 jsCompileList.forEach(function (item) {
                let entryKey = item.path.replace(utils.normalizePath(path.join(global.staticDirectory, 'src/')), 'sf/deploy/').replace('/main.js', '');
                entryList[entryKey] = [
                    'webpack-hot-middleware/client',
                    item.path
                ];
 });
```

dev server端引入webpack.optimize.OccurrenceOrderPlugin()、webpack.HotModuleReplacementPlugin()、webpack.NoEmitOnErrorsPlugin()这三个插件。

```
let config = {
                devtool: "eval",
                entry: entryList,
                plugins: [
                    new webpack.optimize.OccurrenceOrderPlugin(),
                    new webpack.HotModuleReplacementPlugin(),
                    new webpack.NoEmitOnErrorsPlugin()
                ],
                output: {
                    filename: "[name]/bundle.js",
                    chunkFilename: "[name].bundle.js",
                    publicPath: '//' + debugDomain + ':' + global.hotPort + '/'
                }
            };

```

dev-server端引入webpack-dev-middleware和webpack-hot-middleware中间件。

```
app.use(require('webpack-dev-middleware')(compiler, {
                publicPath: config.output.publicPath
            }));

            app.use(require('webpack-hot-middleware')(compiler));
```

之后需要配置转发，将webpack_hmr以及静态资源转发到热替换服务器即可。
fiddler：
>regex:(?insx)^http://localhost:8080/sf/(?<args>.*)$
http://localhost:9797/sf/${args}

>EXACT:http://localhost:8080/__webpack_hmr
http://localhost:9797/__webpack_hmr

charles:

>http://localhost:8080/sf/*  to http://localhost:9797/sf/
>http://localhost:8080/__webpack_hmr to http://localhost:9797/__webpack_hmr

其中8080为本地服务器端口后，9797为热替换服务器端口号, /sf为静态资源目录。详细代码以及使用说明请查看搜狗社区搜索实时开发辅助工具[wenke-dev](https://github.com/mopduan/wenke-dev)。

