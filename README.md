# webpack-demo
webpack4入门配置及实例


2019-5-21：
======

通过babel转化es6为es5语法：

	1.babel-loader只是沟通webpack跟babel之间的桥梁。 @babel/core为babel的核心库 。

	2.babel/preset-env包含了所有es6语法转化为es5语法的规则。

	3.@babe/polyfill解决低版本浏览器不兼容es6的语法。但是polyfill会在解决过程中以全局变量的形式污染全局。
	  所以当我们开发第三类库跟组件的时候需要引入@babel/runtime。他会以闭包的形式引入到我们项目中来

	4.当我们babel-loader的options的内容过多的时候。可以在根目录下创建一个.babelrc的文件。用来放options的内容。
    
    
   [@babel/runtime](https://www.babeljs.cn/docs/babel-plugin-transform-runtime)
   
   
详细配置

    {
        test: /\.js$/,
        exclude: /node_modules/,   // 对于node_modules里面的文件不需要转化
        loader: 'babel-loader',   // 沟通webpack跟babel之间的桥梁。 
        options: {
            presets: [['@babel/preset-env',{    // @babel/preset-env包含了所有es6语法转化为es5语法的规则。
                targets: {
                    chrome: '67',   // chrome67以上的浏览器版本不需要转es5语法
                },
                useBuiltIns: 'usage'    //  @babel/polyfill根据业务代码中出现的es6语法来做打包。减少打包的体积
            }]] 
        }
    }, 



2019-5-22：
======

    react代码用webpack编译示例：

    对于.babelrc配置文件我们配置：(配置文件执行顺序从下往上,从右往左)

    { 
        presets: [
            ['@babel/preset-env',{    // @babel/preset-env包含了所有es6语法转化为es5语法的规则。
                targets: {
                    chrome: '67', // chrome67以上的浏览器版本不需要转es5语法
                },
                useBuiltIns: 'usage'  //  @babel/polyfill根据业务代码中出现的es6语法来做打包。减少打包的体积
            }
            ],
            "@babel/preset-react"   // 对于react的代码。先用preset-react进行编译。然后用preset-env进行es5语法的转换
        ]
    }
    
    
    
2019-5-27：
======
    Tree shaking(过滤掉未通过import导入的模块。)(Tree shaking只支持esmodule模块的导入模式)
    
    开发环境下。需要在webpack.config.js文件下设置:
    modules.exports = {
        mode: 'development',
        optimization: {
           usedExports: true,
        },
    }
    
    
    生产环境下我们只需要将mode改成production就可以了：
    modules.exports = {
       mode: 'production'
    }
    
package.json配置：

    还需要在package.json里面的最上面设置
    "sideEffets": false,   // 如果设置了false就是对所有import引入的模块都进行Tree shaking。但是这样会产生问题。
    
    比如在main.js里面引入了import '@babel/poly-fill' 因为他没有导出一个方法。Tree shaking就会将它自动过滤掉。导致项目打包失败或者报错。
    
    这时候就需要更改配置："sideEffets": ["@babel/poly-fill"]   // 对@babel/poly-fill不进行Tree shaking。这样就会成功了
    
    
    
2019-5-27：
======


开发环境跟生产环境区分打包：

    package.json配置：
    
    "scripts": {
        "dev": "webpack-dev-server --config ./build/webpack.dev.js",  // 开发环境打包配置文件用的是当前目录下build文件夹下的webpack.dev.js
        "build": "webpack --config ./build/webpack.prod.js"  // 生产环境打包配置文件用的是当前目录下build文件夹下的webpack.prod.js
     },
     
     对于两个配置文件相同的部分。我们可以在build目录下创建一个webpack.common.js的文件。用来放dev.js跟prod.js相同的配置代码。
     
     合并导出配置文件的时候通过webpack-merge来合并当前配置文件(dev.js 或者 prod.js)与webpack.common.js的的配置项就可以了




2019-5-28：
======

Code Splitting(代码分割--一般用于更快的加载项目):

    有两种方式：
    
    1、用webpack自带的代码分割的配置项(同步代码分割)
    
    optimization: {      // 代码分割
        splitChunks: {
            chunks: 'all'  // 无论是同步还是异步都分割代码 
        }
    }


    2、 无需任何配置,会自动进行代码分割,放置到新的文件中(异步代码分割)

    例如动态import。动态import的语法是测试中,直接打包会报错,所以需要babel插件babel-plugin-dynamic-import-webpack来转换。安装完打包即可。
    
    
    
    
    
2019-6-3：
======
上节学习了Code Splitting。其实Code Splitting不只可以设置chunks,还可以设置其他参数。这节课就讲一下其他参数
    
    optimization: {      // 代码分割
        splitChunks: {
            chunks: 'all',   // 无论同步还是异步都会进行代码分割。可以设置initial(同步)或者async(异步)
	    
	    	// 以下的配置是默认配置代码。不在webpack.config.js写也可以
            minSize: 0,   // 对于分割出来的代码,如果文件大小 > minSize(30000字节约等于30kb)的时候才会在dist文件夹下生成新的分割文件
            // maxSize: 0,  // 对于分割出来的代码,如果文件大小 > maxSize的话。splitChunks会进行二次分割(了解)
            minChunks: 1,  // 对引用的模块的使用的至少几次的时候才分割
            maxAsyncRequests: 5, // 只分割前5个js文件,多出来的js文件不进行分割
            maxInitialRequests: 3, // 对入口文件,最多分割出3个js文件
            automaticNameDelimiter: '~', // 生成分割文件的连接符号
            name: true, // 可以使用name进行重命名
            cacheGroups: {
                vendors: {
                    test: /[\\/]node_modules[\\/]/,  // 是否从node_modules引入的模块(例如lodash)。是的话走vendors配置。不是走default配置
                    priority: -10,    // 优先级(越大优先级越高)
                    name: 'vendors.js', // 重命名
                },
                default: {
                    priority: -20,
                    reuseExistingChunk: true, // 如果一个模块被打包过了。那么再次打包的时候,不会被打包。会直接使用这个模块
                    name: 'common.js', // 重命名
                },
            }
        }
    },



2019-6-5：
======
    今天主要学习了几种依赖收集插件(建议用webpack-bundle-analyzer)来生成依赖树。然后打包之后会出现stats.json。把他丢到插件里就可以生成试图了
    
    
   [webpack-bundle-analyzer](https://github.com/webpack-contrib/webpack-bundle-analyzer)

    还有Prefetch跟Preload来提前下载资源。优化页面加载速度。但是两者会有区别
    
   [Prefetch/Preload](https://webpack.js.org/guides/code-splitting#prefetchingpreloading-modules)
    
    1.Prefetch会在核心代码加载完成之后。页面有空闲的时候才会加载。以便加快后续页面的首屏速度
    
      (Prefetch 加载的资源一般不是用于当前页面的，即未来很可能用到的这样一些资源，简单点说就是其他页面会用到的资源)
    
    2.Preload则是会跟核心代码一起加载。
    
      (preload主要用于预加载当前页面需要的资源)
      
