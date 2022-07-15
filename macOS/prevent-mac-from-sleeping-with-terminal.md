# 터미널에서 맥이 잠들지 않게 하기

- 실행방법 : 터미널에서 아래 명령을 실행한다.
- `caffeinate` 만 실행해도 되지만 모니터 꺼짐도 방지하기 위해 `-d` 옵션을 추가한다.
  ```shell
  $ caffeinate -d
  ```

- 시간 단위 지정은 -t 옵션 (3600초 - 1시간, 21600초 - 6시)
  ```
  caffeinate -t 3600
  ```

- 상세 옵션은 `man caffeinate` 에서 확인 가능

# References
- https://ss64.com/osx/caffeinate.html
- https://seorenn.tistory.com/61
