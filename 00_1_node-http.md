# Node and HTTP

We can open a socket connection and handle inbound requests.

1. The method `createServer()` of the http interface will invoke logic in the C++ side of NodeJS to handle network features, allowing us to specify some options (like the port).
2. The C++ features are going to open a socket connection (with the help of libuv, which will handle the logic for each OS) on the device.
3. Every time a client makes an HTTP request to the server, the C++ side is going to instruct JS to execute the JS logic that we are passing to createServer() method.
4. With every new request, Node will provide `req` and `res` objects

```js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('okay');
});

server.listen(9090);
```


In the previous example, a client (like your web browser) is going to make an HTTP request to the server.
The server (NodeJS) will respond with a message (in this case, "okay").


