# Elasticsearch 에서 JSON 문자열을 JSON 객체 필드로 추가하기 (feat.pipeline)

Elasticsearch 에서 기존 필드중에 JSON 문자열을 그대로 저장하는 필드가 있다.

---
### 변경전

- Field
  - name : sites 
  - type : text
  - value : JSON 문자열
    ```
    { 
      "google": "search",
      "facebook": "social",
      "twitter": "social" 
    }
    ```
- Elasticsearch 에 저장된 값
 
  ```shell
  "sites": "{\"google\":\"search\",\"facebook\":\"social\",\"twitter\":\"sosial\"}"
  ```
- _analyze API 요청값
  ```
  $ _analyze
  {
    "text" : "{\"google\":\"search\",\"facebook\":\"social\",\"twitter\":\"sosial\"}",
  }'

- _analyze API 반환값
  ```
  "tokens" : [
    { "token" : "google" ... },
    { "token" : "search" ... },
  ]
  ```

- 화면에 보여주기 위해 가져올 때는 JSON.Parse 처리등을 해서 보여줄 수 있다.
- 하지만, 검색을 위해서(`google이 search 값인지`) 는 문자열로 원하는 결과를 얻기가 어렵다.
  - Standard Analyzer 의 Token 처리방식에 의해 `google: search` 값을 뽑아내기가 어렵다. [참고](https://idea-sketch.tistory.com/64)
  - (정규표현식으로 어떻게든 뽑아낼 수 있겠지만 보다 명확한 방법을 선호)  

---

### 변경후

- [Object field type](https://www.elastic.co/guide/en/elasticsearch/reference/master/object.html) 을 사용한다. (JSON object)
- Elasticsearch 저장할 때 `JSON.parse()` 처리하여 신규필드에 JSON Object 로 저장 
- 예시에서는 `sites_json` field 추가, 별도의 맵핑 정의가 없다면 ES 는 필드를 값에 따라 자동 정의하여 생성함)
- 검색예시
  ```
  $ _search
  '{
    "query": {
      "bool": { 
        "filter": [
          { "term": { "sites_json.google": "search" } }
        ]
      }
    }
  }
  ```
- Mission, Success. 필터링 검색이 가능해졌다. 😎

---

### 기존 데이터의 업데이트처리

- 신규 등록되는 데이터는 JSON Object 필드로 저장하지만, 기존 데이터들은 업데이트 해야 한다.
- Elasticsearch 에서는 Ingest Pipeline 을 통한 전처리 프로세싱을 지원한다. 
  - [Ingest Pipeline 참고](https://danawalab.github.io/elastic/2020/09/04/ElasticSearch-IngestPipeLine.html)
- [JSON processor](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/json-processor.html) 를 사용해 기존 데이터의 문자열을 JSON 으로 변환한다.

- 우선은, 잘 되는지 혹시 모르니 먼저 시뮬레이트
  ```
  POST _ingest/pipeline/_simulate 
  {
    "pipeline": {
      "description": "JSON string to JSON object",
      "processors": [{
        "json": {
          "field": "sites",
          "target_field": "sites_json"
        } 
        }
      ]
    },
    "docs": [
      {
        "_source": {
          "sites" : "{\"google\":\"search\",\"facebook\":\"social\",\"twitter\":\"sosial\"}"
        }
      }
    ]
  }

  결과
  {
    "docs" : {
      ...
      "_source": {
        "sites" : "{\"google\":\"search\",\"facebook\":\"social\",\"twitter\":\"sosial\"}",
        "sites_json": { 
          "google": "search",
          "facebook": "social",
          "twitter": "social" 
        }
      }
      ...
    }
  }
  ```

- simulate 해보니 잘 된다. JSON Object 로 원하는 결과 출력 확인.
- 이제 JSON processor 를 사용하는 pipeline 등록
  ```
  PUT _ingest/pipeline/sites_json_pipeline 
  {
    "description": "JSON string to JSON object",
    "processors": [{
      "json": {
        "field": "sites",
        "target_field": "sites_json"
      }
    }]
  }
  ```

- 끝으로 pipeline 으로 데이터 업데이트 
  - (당연하지만, 데이터가 많으면 오래 걸린다. m4.large 기준 약 2시간동안 cpu 100% 가까이 치솟았다. 하지만 업댓중에도 검색은 정상작동함)
  - (게다가 Time out 응답이 오지만, 최종적으로 확인해 보면 정상 업데이트되어 있다.)
    ```
    POST my-data/_update_by_query?pipeline=sites_json_pipeline
    ```

- 난 pipeline 으로 JSON processor 만 사용했지만 다양한 ‌Ingest pipeline processor 가 존재한다.
- 예) HTML strip processor, Split processor 등
- pipeline 을 default 로 설정해 데이터 전처리만으로도 등록되는 데이터들마다 바로 JSON 변환이 가능하지만, 코드에 명시해서 쓰는 편이 좀 더 명확해서 그렇게 하진 않음.
