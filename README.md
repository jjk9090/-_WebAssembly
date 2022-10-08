# -_WebAssembly
* 初探WebAssembly

  - 基本要求：

    - 使用WebAssembly技术的代码

    - 编写index.html

      - ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
          <meta charset="UTF-8">
          <meta http-equiv="X-UA-Compatible" content="IE=edge">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <title>Document</title>
        </head>
        <body>
          <script>
            fetch('./test.wasm').then(Response =>
              Response.arrayBuffer()
            ).then(bytes =>
              WebAssembly.compile(bytes)
            ).then(mod =>{
              const instance = new WebAssembly.Instance(mod)
              const a = instance.exports
              console.log(a);
              console.time("测试fib 速度:");
              var re = a._Z3fibi(40);
              console.timeEnd("测试fib 速度:");
            }
            );
          </script>
        </body>
        </html>
        ```

        - 用WebAssembly Explorer在线将c程序编写成.wasm文件
        - 在powershell 用live-server --potr=1111打开，在控制台查看得到

    - 纯javasript

      - 编写test.html

      - ```html
        <!DOCTYPE html>
        <html lang="en">
        <head>
          <meta charset="UTF-8">
          <meta http-equiv="X-UA-Compatible" content="IE=edge">
          <meta name="viewport" content="width=device-width, initial-scale=1.0">
          <title>Document</title>
        </head>
        <body>
          <script>
            function fib(x){
              if(x<=0)
                return 0;
              if(x<=2)
                return 1;
              return fib(x-1)+fib(x-2);
            }
            console.time("测试fib 速度:");
            var re = fib(40);
            console.timeEnd("测试fib 速度:");
            
          </script>
        </body>
        </html>
        ```

  - 可选项

    - 编写node.js文件

    - ```js
      const http = require('http');
      const url = require('url');
      const fs = require('fs');
      const path = require('path');
      
      const PORT = 8888;  // 服务器监听的端口号；
      
      const mime = {
        "html": "text/html;charset=UTF-8",
        "wasm": "application/wasm"  //当遇到对".wasm"格式文件的请求时，返回特定的MIME头；
      };
      
      http.createServer((req, res) => {
        let realPath = path.join(__dirname, `.${url.parse(req.url).pathname}`);
        //检查所访问文件是否存在，且是否可读；
        fs.access(realPath, fs.constants.R_OK, err => {  
          if (err) {
            res.writeHead(404, { 'Content-Type': 'text/plain' });
            res.end();
          } else {
            fs.readFile(realPath, "binary", (err, file) => {
              if (err) {
                //文件读取失败时返回500；
                res.writeHead(500, { 'Content-Type': 'text/plain' });
                res.end();
              } else {
                //根据请求的文件返回相应的文件内容；
                let ext = path.extname(realPath);
                ext = ext ? ext.slice(1) : 'unknown';
                let contentType = mime[ext] || "text/plain";
                res.writeHead(200, { 'Content-Type': contentType });
                res.write(file, "binary");
                res.end();
              }
            });
          }
        });
      }).listen(PORT);
      console.log("Server is runing at port: " + PORT + ".");
      ```

    - 编写index.html

    - ```html
      <!DOCTYPE html>
      <html lang="en">
        <head>
          <meta charset="UTF-8" />
          <meta name="viewport" content="width=device-width, initial-scale=1.0" />
          <title>斐波纳切数字</title>
        </head>
        <script>
          function fibonacciJS(n) {
            if (n < 2) {
              return 1;
            }
            return fibonacciJS(n - 1) + fibonacciJS(n - 2);
          }
          const response = fetch("fibonacci.wasm");
          const num = [5, 15, 25, 35, 45];
          WebAssembly.instantiateStreaming(response).then(
            ({ instance }) => {
              let { fibonacci } = instance.exports;
              for(let n of num) {
                console.log(`斐波纳切数字: ${n}，运行 10 次`)
                let cTime = 0;
                let jsTime = 0;
                for(let time = 0; time < 10; time++) {
                  let start = performance.now();
                  fibonacci(n)
                  cTime += (performance.now() - start)
      
                  start = performance.now();
                  fibonacciJS(n)
                  jsTime += (performance.now() - start)
                }
                console.log(`wasm 模块平均调用时间：${cTime / 10}ms`)
                console.log(`js 模块平均调用时间：${jsTime / 10}ms`)
              }
      
            }
          )
        </script>
        <body>
        </body>
      </html>
      ```

    - 用node node.js打开，则在http://localhost:8888/index.html 打开控制台
