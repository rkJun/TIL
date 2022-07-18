# Visual Studio Code 에서 markdown 파일만 trimTrailingWhitespace 옵션 끄기

소스 코드의 불필요한 후행 공백 제거를 위해 
    `"files.trimTrailingWhitespace": true` 옵션을 사용하고 있다.
하지만 markdown 파일의 경우에는 후행 공백 2칸으로 개행 처리를 한다.  
그러므로, markdown 만 해당 옵션을 `false` 로 설정한다.

settings.json 파일에서 
```
  "files.trimTrailingWhitespace": true,
  "[markdown]": {
      "files.trimTrailingWhitespace": false
  },
```

# References
- https://github.com/microsoft/vscode/issues/1679
- https://velog.io/@gidskql6671/Vscode-Markdown%EC%97%90%EC%84%9C%EB%A7%8C-Trim-Trailing-Whitespace%EC%98%B5%EC%85%98%EC%9D%84-%EB%81%84%EA%B8%B0
