# express-session

- Express 의 Session 처리를 위해 [express-session](https://github.com/expressjs/session)을 사용한다.
- Session은 Redis 에 보관한다. [connect-redis](https://github.com/tj/connect-redis) 사용
  - Redis 의 TTL (Time To Live) 은 초단위로 사용한다.
    - 예) `60 * 60 * 24`  = `1 day`

- Redis 서버의 Session 과 짝을 이루는 브라우저의 cookie 값은 별도로 `name` 을 변경하지 않으면 기본값 `connect.sid` 을 사용한다.
- `cookie.maxAge` 를 사용하여 브라우저 cookie 의 만료기한을 설정할 수 있다.
  - 밀리초단위라 `1000` 이 1초와 같다.
    - 예) `1000 * 60 * 60 * 24` = `1 day`
      ```
      app.use(session({
        cookie: {
          maxAge: 1000 * 60 * 60 * 24 
        },
      ```
- 서버의 세션값과 브라우저 쿠키에 저장된 세션값이 동일하고, 둘 다 유효한 상태여야 로그인이 유지되므로, 둘중 한곳의 세션이 만료되거나 삭제되면 로그아웃된다.
  1. 서버세션 만료 - 서버 세션의 TTL 시간 동안 서버와 브라우저간 호출응답이 발생하지 않은 경우 서버 세션은 만료된다.
     (호출시 서버세션은 설정된 TTL 로 지속적으로 갱신하므로 TTL 내에서 계속 요청/응답을 한다면 만료하지 않음)
  2. 브라우저의 쿠키에 저장된 세션 만료 - `cookie.maxAge` 에 따라 설정된 만료기한이 지나면 삭제된다.

- 브라우저의 쿠키 만료를 신경쓰고 싶지 않다면 `rolling: true` 옵션을 사용한다. [참고](https://github.com/expressjs/session#rolling) 
  - 브라우저쪽은 자동 갱신하고, 서버 세션만으로 유지한다.
    ```
    app.use(session({
      cookie: {
        rolling: true
      },
    ```