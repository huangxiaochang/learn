搭建本地https服务器
==================

使用nodejs + express来搭建，所以需要先安装nodejs。

## 使用openssl生成https证书。
1.安装openssl。
2.进入bin目录，进行cmd.
3.生成私钥文件,输入以下命令：
```
openssl genrsa -out privatekey.pem 1024
```
4.通过私钥生成CSR证书签名：
```
openssl req -new -key privatekey.pem -out certrequest.csr
// 在这一步需要输入国家，省，市等信息，如：
/*
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:Beijing
Locality Name (eg, city) []:Beijing
Organization Name (eg, company) [Internet Widgits Pty Ltd]:fens.me
Organizational Unit Name (eg, section) []:fens.me
Common Name (eg, YOUR name) []:Conan Zhang
Email Address []:bsspirit@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
*/
```
5.通过私钥和证书签名生成证书文件
```
openssl x509 -req -in certrequest.csr -signkey privatekey.pem -out certificate.pem
```
6.然后再bin目录下会生成三个文件：
privatekey.pem: 私钥
certrequest.csr: CSR证书签名
certificate.pem: 证书文件

## 搭建express服务器

```
// app.js
const express = require('express')
const path = require('path')
const http = require('http')
const fs = require('fs')
const https = require('https')

// const router = express.Router()

const app = express()

app.use(express.static(path.join(__dirname, 'public')))
app.set('public', __dirname)

app.set('view engine', 'html')
app.engine('.html', require('ejs').__express)

// 允许跨域
app.all('*', function(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Content-Type, Content-Length, Authorization, Accept, X-Requested-With");
    res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
    res.header("X-Powered-By",' 3.2.1')
    if(req.method=="OPTIONS") res.send(200);
    else  next();
})

app.get('/', function(req, res) {
  if (req.protocol === 'https') {
    res.render('https');
  } else {
    res.render('index');
  }
})

app.get('/index', function(req, res) {
  res.render('a');
})

app.post('/postData', function (req, res) {
  req.on('data',function(data){
    obj = JSON.parse(data);
    console.log(obj);
    let msg = req.protocol === 'https' ? 'https操作成功' : 'http操作成功';
    res.json(200, { msg: msg, success: 1 })
  })
})

app.get('/getUser', function(req, res, next) {
  console.log('getUser', req.data)
  res.json(200, { msg: '操作成功', success: 1 })
});


//根据项目的路径导入生成的证书文件,该证书文件为上一步使用openssl生成的证书文件。
const privateKey = fs.readFileSync('privatekey.pem');
// const privateKey  = fs.readFileSync(path.join(__dirname, './private.pem'), 'utf8');
const certificate = fs.readFileSync('cert.pem');
// const certificate = fs.readFileSync(path.join(__dirname, './ca.cer'), 'utf8');
const credentials = {key: privateKey, cert: certificate};

const httpServer = http.createServer(app)
httpServer.listen(8085, function () {
  console.log('server running in: http://localhost:8085')
})

const httpsServer = https.createServer(credentials, app);

httpsServer.listen(3001, function () {
  console.log('server running in: https://localhost:3001')
});

```
执行node app.js即可启动服务器，然后通过https://localhost:3001即可看到相应的页面内容