# Base64 인코딩과 디코딩

## Base64 란
 > 컴퓨터 분야에서 쓰이는 **Base 64** (베이스 육십사)란 8비트 이진 데이터(예를 들어 [실행 파일](https://ko.wikipedia.org/wiki/%EC%8B%A4%ED%96%89_%ED%8C%8C%EC%9D%BC "실행 파일")이나, [ZIP](https://ko.wikipedia.org/wiki/ZIP "ZIP") 파일 등)를 [문자 코드](https://ko.wikipedia.org/wiki/%EB%AC%B8%EC%9E%90_%EC%BD%94%EB%93%9C "문자 코드")에 영향을 받지 않는 공통 ASCII 영역의 문자들로만 이루어진 일련의 문자열로 바꾸는 [인코딩](https://ko.wikipedia.org/wiki/%EC%9D%B8%EC%BD%94%EB%94%A9 "인코딩") 방식을 가리키는 개념이다.
 >
>원래 **Base 64**를 글자 그대로 번역하여 보면 64진법이란 뜻이다. 특별히 64진법이 컴퓨터에서 흥미로운 것은, 64가 2의 제곱수(64 = 26)이며, 2의 제곱수들에 기반한 진법들 중에서 화면에 표시되는 [ASCII](https://ko.wikipedia.org/wiki/ASCII "ASCII") 문자들을 써서 표현할 수 있는 가장 큰 진법이기 때문이다. 즉, 다음 제곱수인 128진법에는 128개의 기호가 필요한데 화면에 표시되는 ASCII 문자들은 128개가 되지 않는다.
>
>그런 까닭에 이 인코딩은 전자 메일을 통한 이진 데이터 전송 등에 많이 쓰이고 있다. Base 64에는 어떤 문자와 기호를 쓰느냐에 따라 여러 변종이 있지만, 잘 알려진 것은 모두 처음 62개는 알파벳 A-Z, a-z와 0-9를 사용하고 있으며 마지막 두 개를 어떤 기호를 쓰느냐의 차이만 있다.

[위키백과 참고](https://ko.wikipedia.org/wiki/%EB%B2%A0%EC%9D%B4%EC%8A%A464)

- 간단히 말해서 Base64 Encoding은 Binary Data를 Text로 변경하는 Encoding
- Base64 Encoding을 하게 되면 전송해야 될 데이터의 양도 약 33% 정도 늘어남. 
	- 6bit당 2bit의 Overhead가 발생하기 때문. 
- Encoding전 대비 33%나 데이터의 크기가 증가하고, Encoding과 Decoding에 추가 CPU 연산까지 필요한데 우리는 왜 Base64 Encoding을 하는가?
	- 문자를 전송하기 위해 설계된 Media(Email, HTML)를 이용해 플랫폼 독립적으로 Binary Data(이미지나 오디오)를 전송 할 필요가 있을 때, ASCII로 Encoding하여 전송하게 되면 여러가지 문제가 발생할 수 있다. 대표적인 문제
		- ASCII는 7 bits Encoding인데 나머지 1bit를 처리하는 방식이 시스템 별로 상이.
		- 일부 제어문자 (e.g. Line ending)의 경우 시스템 별로 다른 코드값을 갖는다.
	- Base64는 HTML 또는 Email과 같이 문자를 위한 Media에 Binary Data를 포함해야 될 필요가 있을 때, 포함된 Binary Data가 시스템 독립적으로 동일하게 전송 또는 저장되는걸 보장하기 위해 사용.

## JavaScript 에서 Base64 인코딩, 디코딩
- Base64 로 인코딩
```
btoa(stringToEncode)
```
- Base64 로 디코딩
```  
atob(encodedData)
```

### 예제 
```
> btoa('test')
'dGVzdA=='
> atob('dGVzdA==')
'test'
```

### Base64 인코딩이 안되는 경우
- 크롬 에러 예)
```
> btoa('한글')
VM258:1 Uncaught DOMException: Failed to execute 'btoa' on 'Window': The string to be encoded contains characters outside of the Latin1 range.
    at <anonymous>:1:1
```
- 노드 에러 예)
```
> btoa('한글')
Uncaught:
DOMException [InvalidCharacterError]: Invalid character
    at new DOMException (node:internal/per_context/domexception:53:5)
    at __node_internal_ (node:internal/util:502:10)
    at btoa (node:buffer:1232:13)
```

- 해결방법
```
btoa(unescape(encodeURIComponent(str)))
```

- 해당 문자열을 `encodeURIComponent` 처리후, `unescape` 처리한다.
(`unescape` 는 하지 않아도 되지만, 그만큼 문자열이 길어지므로 사용하여 인코딩된 문자열을 줄이도록 한다)

- 결과 예)
```
> encodeURIComponent('한글')
'%ED%95%9C%EA%B8%80'
> encodeURIComponent('한글').length
18

> btoa(encodeURIComponent('한글'))
'JUVEJTk1JTlDJUVBJUI4JTgw'
> btoa(encodeURIComponent('한글')).length
24

> unescape(encodeURIComponent('한글'))
'í\x95\x9Cê¸\x80'

> unescape(encodeURIComponent('한글')).length
6

> btoa(unescape(encodeURIComponent('한글')))
'7ZWc6riA'

> btoa(unescape(encodeURIComponent('한글'))).length
8
```

## References
- https://ko.wikipedia.org/wiki/%EB%B2%A0%EC%9D%B4%EC%8A%A464
- https://effectivesquid.tistory.com/entry/Base64-%EC%9D%B8%EC%BD%94%EB%94%A9%EC%9D%B4%EB%9E%80
- https://webisfree.com/2017-04-25/javascript%EC%97%90%EC%84%9C-%EB%AC%B8%EC%9E%90%EB%A5%BC-base64%EB%A1%9C-%EC%9D%B8%EC%BD%94%EB%94%A9-%EB%B3%80%ED%99%98%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95%EC%9D%80
- https://webisfree.com/2017-04-25/javascript%EC%97%90%EC%84%9C-%EB%AC%B8%EC%9E%90%EB%A5%BC-base64%EB%A1%9C-%EC%9D%B8%EC%BD%94%EB%94%A9-%EB%B3%80%ED%99%98%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95%EC%9D%80
- https://developer.mozilla.org/ko/docs/Web/API/btoa
- https://developer.mozilla.org/ko/docs/Web/API/atob