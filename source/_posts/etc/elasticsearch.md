---
title: 'elasticsearch'
date: 2019-01-25 18:24:43
tags:
---

참고자료  
<https://sanghaklee.gitbooks.io/elk/content/>  

elk docker로 띄우기
https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html

ELk를 마스터하면 어떤 빅데이터를 만나든 쉽게 다룰 수 있다?
로그 스테이시가 어떤 데이터든(csv든 상관없이) 수집해서 엘라스틱 서치에 수집해주고,
키바나는 데이터 비주얼라이션 툴로써 엘라스틱 서치의 데이터를 화면에 보기좋게 보여줌

엘리스틱 서치 : 키워드가 어떤 document에 있다고 저장하는 식이다
인덱스(가장 큰 개념)안에 타입, 타입안에 도큐먼트

REST API로 데이터 CRUD를 수행한다

인덱스 생성은 PUT으로 하고, document 생성은 POST로 한다
json 파일을 직접 넣을 수 있다

값 업데이트 : _update
_update시에 script에 객체탐색형식으로 접근해서 값을 수정할 수 있다

bulk insert : _bulk
bulk는 2개의 라인으로 구성되어있다
1번 라인은 metadata, 2번 라인은 데이터
데이터 라인에서 기본적인 데이터 타입은 자동으로 잡아서 mappings로 등록해주는 듯 하다

{ "index" : { "_index" : "basketball", "_type" : "record", "_id" : "1" } }
{"team" : "Chicago Bulls","name" : "Michael Jordan", "points" : 30,"rebounds" : 3,"assists" : 4, "submit_date" : "1996-10-11"}
{ "index" : { "_index" : "basketball", "_type" : "record", "_id" : "2" } }
{"team" : "Chicago Bulls","name" : "Michael Jordan","points" : 20,"rebounds" : 5,"assists" : 8, "submit_date" : "1996-10-11"}

이런 데이터를 등록하면 자동으로 mappings에. 
"mappings" : {
      "record" : {
        "properties" : {
          "assists" : {
            "type" : "long"
          },
          "name" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          },
          "points" : {
            "type" : "long"
          },
          "rebounds" : {
            "type" : "long"
          },
          "submit_date" : {
            "type" : "date"
          },
          "team" : {
            "type" : "text",
            "fields" : {
              "keyword" : {
                "type" : "keyword",
                "ignore_above" : 256
              }
            }
          }
        }
      }
식으로 등록이 되어있었다.

엘라스틱서치에 데이터매핑을 안할수 있긴 하지만 위험하다. (키바나 시각화 등)
curl -XGET http://localhost:9200/classes?pretty 하면 mappings가 나오는데, 각 필드별 타입등을 지정해놓은 구간이다
`classes/class/_mapping -d @json파일` 형태로 지정 가능
이후 bulk insert하면 잘 들어간것을 확인할 수 있다는데, mapping이 있고 없고 차이가 뭔지?

curl -XGET http://localhost:9200/{_index}/{_type}/_search?q=points:30 
requestBody로도 검색 가능

<!-- more -->