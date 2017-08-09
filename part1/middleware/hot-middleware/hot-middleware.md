### wepack-hot-middleware 深入解读

1. webpack-hot-middleware 做什么的？
   webpack-hot-middleware 是和webpack-dev-middleware配套使用的。从上一篇文章中知道，webpack-dev-middleware是一个express中间件，实现的效果两个，一个是在fs基于内存，提高了编译读取速度；第二点是，通过watch文件的变化，动态编译。但是，这两点并没有实现浏览器端的加载，也就是，只是在我们的命令行可以看到效果，但是并不能在浏览器动态刷新。那么webpack-hot-middleware就是完成这件小事的。没错，这就是一件小事，代码不多，后面我们就深入解读。
2. webpack-hot-middleware 怎么使用的？
   [官方文档](https://github.com/glenjamin/webpack-hot-middleware)已经很详细的介绍了，那么我再重复一遍。
   1. 在plugins中增加HotModuleReplacementPlugin().
   2. 在entry中新增webpack-hot-middleware/client
   3. 在express中加入中间件webpack-hot-middleware.
   详细的配置文件也可以看[我的github](https://github.com/webfrontzhifei/webpack-step-step/tree/master/part1/middleware/hot-middleware/example)
   ```js
   // webpack.config.js
   const path = require('path');
   const webpack = require('webpack');
   module.exports = {
       entry: ['webpack-hot-middleware/client.js', './app.js'],
       output: {
           publicPath: "/assets/",
           filename: 'bundle.js',
           //path: '/'   //ֻʹ�� dev-middleware ���Ժ��Ա�����
       },
       plugins: [
           new webpack.HotModuleReplacementPlugin(),
           new webpack.NoEmitOnErrorsPlugin()   //����ʱֻ��ӡ���󣬵������¼���ҳ��
       ]
   };
   // express.config.js
   app.use(require("webpack-hot-middleware")(compiler, {
       path: '/__webpack_hmr',
   }));

   app.get("/", function(req, res) {
       res.render("index");
   });

   app.listen(3333);
   ```
3. 重点，重点，重点，源代码分析。
   通过上面的配置，我们可以发现webpack实现了热加载，那么是如何实现的呢？进入正题。
   webpack-hot-middleware中的入口文件middleware.js，哇，只有100多行。