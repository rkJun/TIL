# Top Level await

- 비동기 호출을 위한 async/await 사용시에
  - 모듈의 최상위 레벨에서 사용하기 위해서는 
  - await 사용을 위해서 반드시 async 가 수반되어야 한다.
  - 그러다 보니 바로 쓰더라도 일부러 async 로 감싸서 실행시키는 방법을 쓰거나
  ```
  (async () => {
    const response = await doFunc();
    console.log(response);
  })();
  ```
  - async function 을 정의한 후 호출해 쓰거나 했는데...
- 이제 모듈의 최상위 레벨에서 async 없이도 바로 await 사용이 가능하다.
  - 비동기 모듈 import 시에 사용한다거나, `await import('/path/module');`
  - 내 경우는 node-redis 를 v3 에서 v4 마이그레이션 할 때 사용했다.
    - node-redis 가 더이상 Auto Connect 를 지원하지 않아, connect() 를 호출해 주어야함. 
    - [참고](https://github.com/redis/node-redis/blob/master/docs/v3-to-v4.md)
    ```
    const client = createClient();
    await client.connect();
    ```
  - es2022 에서 표준으로 지정됨
  - [참고](https://www.ecma-international.org/publications-and-standards/standards/ecma-262/)
- Node.js 에서는 v14.8 부터 (별도의 flag 옵션없이) 사용이 가능했다.
   - [참고](https://www.stefanjudis.com/today-i-learned/top-level-await-is-available-in-node-js-modules/)