Node JS의 스트림에 대해
===================

Node JS를 그저 도큐멘트에서 쓰여진 예제를 수정하는 정도로 쓰다가 클라이언트 -> 서버로 
파일을 전송하는 부분에 대해 궁금해지던 중 미뤄뒀던 Stream에 대해 공부해야겠다고 생각했다.

[Node.js Stream 당신이 알아야 할 모든것](https://github.com/FEDevelopers/tech.description/wiki/Node.js-Stream-%EB%8B%B9%EC%8B%A0%EC%9D%B4-%EC%95%8C%EC%95%84%EC%95%BC%ED%95%A0-%EB%AA%A8%EB%93%A0-%EA%B2%83)이라는 문서와
[노드JS공식문서 - HTTP 트랜젝션 해부](https://nodejs.org/ko/docs/guides/anatomy-of-an-http-transaction/)를 봤다.

먼저 node.js의 request객체는 ReadableStream이고 response객체는 WritableStream이다.
그리고 ReadableStream과 WritableStream을 포함한 모든 스트림은 EventEmitter이다.

아래 코드는 readable스트림을 직접 이벤트와 함께 사용한 코드이다.
data이벤트를 통해 계속 들어오는 스트림을 받을 수 있다.
```
readable.on('data', chunk => {
    writable.write(chunk)
})
readable.on('end', () => {
    writable.end()
})
```

위에서 request는 객체는 ReadableStream이라고 했다.
이 부분을 request객체에 적용시킨 코드가 Nodejs 공식문서의 body를 parse하는 부분이다.

```
let body = [];
request.on('data', chunk => {
    body.push(chunk);
}).on('end', () => {
    body = Buffer.concat(body).toString()
})
```

자 서버로 들어온 요청을 읽어들였으니 무언가를 다시 리턴해야한다.
response객체는 WritableStream이라고 했다.

먼저 WritableStream의 기본적인 형태를 한번보자
```
const { Writable } = require('stream')

const outStream = new Writable({
    write(chunk, encoding, callback){
        console.log(chunk.toString());
    }
})
process.stdin.pipe(outStream)
```
outStream객체를 Writable생성자를 통해 만들면서 process.stdin을 통해 전달된 청크를 단순히 에코하는 write함수를 작성했다. write를 수정하면 stdin을 통해 들어온 청크를 가공할 수 있다.

response객체는 스트림 메서드인 write를 통해 아래와 같이 작성 할 수 있다.
```
response.write('<html>');
response.write('<body>');
response.write('<h1>Hello, World!</h1>');
response.write('</body>');
response.write('</html>');
response.end();
```

위 내용을 기반으로 단순 에코 서버를 작성할때 아래와 같은 코드가 될것이다.
```
const http = require('http');

http.createServer((request, response) => {
  let body = [];
  request.on('data', (chunk) => {
    body.push(chunk);
  }).on('end', () => {
    body = Buffer.concat(body).toString();
    response.end(body);
  });
}).listen(8080);
```