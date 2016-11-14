# ELK를 이용해서 서울시 지하철 대시보드 만들기

[동영상](https://www.youtube.com/watch?v=xPjNtd8xUZo)
[참조문서](https://github.com/kimjmin/elastic-demo/blob/master/doc/201604_webinar.md)

## Elastic Search 설치(2.4.x)

- https://www.elastic.co 에서 download
- 압축 해제. `~/lib/elk $ tar xvfz ~/Downloads/elasticsearch-2.4.1.tar.gz'
- `ES_MIN_MEM=512m bin/elasticsearch`를 실행시킴
- http://localhost:9200/ 에서 확인

## Kibana 설치(4.x)

- https://www.elastic.co 에서 download
- 압축 해제. `~/lib/elk $ tar xvfz ~/Downloads/kibana-4.6.2-darwin-x86_64.tar.gz`
- `bin/kibana`를 실행시킴
- http://localhost:5601/ 에서 확인

## Elastic Search Index 생성

https://github.com/kimjmin/elastic-demo 의 `매핑 정보 입력`을 참고하여 아래와 같이 curl 명령 실행

```
~/lib/elk/kibana-4.6.2-darwin-x86_64 $ curl -XPUT 'http://127.0.0.1:9200/seoul-metro-2014' -d '
  {
    "mappings" : {
      "seoul-metro" : {
        "properties" : {
          "time_slot" : { "type" : "date" },
          "line_num" : { "type" : "string", "index" : "not_analyzed" },
          "line_num_en" : { "type" : "string", "index" : "not_analyzed" },
          "station_name" : { "type" : "string", "index" : "not_analyzed" },
          "station_name_kr" : { "type" : "string", "index" : "not_analyzed" },
          "station_name_en" : { "type" : "string", "index" : "not_analyzed" },
          "station_name_chc" : { "type" : "string", "index" : "not_analyzed" },
          "station_name_ch" : { "type" : "string", "index" : "not_analyzed" },
          "station_name_jp" : { "type" : "string", "index" : "not_analyzed" },
          "station_geo" : { "type" : "geo_point" },
          "people_in" : { "type" : "integer" },
          "people_out" : { "type" : "integer" }
        }
      }
    }
  }'
{"acknowledged":true}%
```

아래와 같이 GET 명령을 통해 인덱스 생성 상태를 확인

```
~/lib/elk/kibana-4.6.2-darwin-x86_64 $ curl -XGET 'http://127.0.0.1:9200/seoul-metro-2014/_search'
{"took":2,"timed_out":false,"_shards":{"total":5,"successful":5,"failed":0},"hits":{"total":0,"max_score":null,"hits":[]}}%
```

## Logstash 설치(2.4.x)

- https://www.elastic.co 에서 download
- 압축 해제. `~/lib/elk $ tar xvfz ~/Downloads/logstash-2.4.0.tar.gz`

### FileBeat(1.3.x)
- https://www.elastic.co 에서 download
- 압축 해제. `~/lib/elk $ tar xvfz   ~/Downloads/filebeat-1.3.1-darwin.tgz`

## Data Loading하기

- https://github.com/kimjmin/elastic-demo 에서 맨 마지막의 `1.4 추출된 데이터 링크`에서 파일을 다운받고, 압축을 푼다(ex. 여기서는 ~/lib/elk/data 디렉토리에 저장)
- filebeats.yml 수정
	- paths 수정해서 `/Users/msbaek/lib/elk/data/*.log` 를 사용하도록 변경
    ![](https://api.monosnap.com/rpc/file/download?id=1CbihXn2du5Vx82sLqSEC78hKCXDSe)
	- output: 섹션에서 elasticsearch를 comment out
    ![](https://api.monosnap.com/rpc/file/download?id=80R1YubGUfe6eRT0y8RggUOxJGuJon)
	- output: 섹션에서 logstash를 enable(hosts도 enable)
    ![](https://api.monosnap.com/rpc/file/download?id=ZLGvacOwaz8ofoH2rbEwtvP9gIrvsZ)
- `~/lib/elk/logstash-2.4.0/logstash.conf` 파일을 아래와 같이 작성

    ```language=java
    input {
      beats {
        codec => json
        port => 5044
      }
    }

    filter {
      mutate {
        remove_field => [ "@version", "@timestamp", "host", "path" ]
      }
    }

    output {
      stdout {
        codec => json
      }
      elasticsearch {
        hosts => ["127.0.0.1"]
        index => "seoul-metro-2014"
        document_type => "seoul-metro"
      }
    }
    ```
- logstash 실행: `~/lib/elk/logstash-2.4.0 $ bin/logstash -f logstash.conf`
- filebeat 실행: `~/lib/elk/filebeat-1.3.1-darwin $ ./filebeat -c filebeat.yml`
	![](https://api.monosnap.com/rpc/file/download?id=Kj2cc77IZt8xnGHUbvpdv2pGUR2Ncp)
    
## Kibana Visualize

### Line Chart

visualize / new visualization / line chart / from a new search

![](https://api.monosnap.com/rpc/file/download?id=3DqEKXEzT6xEGAYwkvTWvLKjD9XLP4)

### Pie Chart

![](https://api.monosnap.com/rpc/file/download?id=NLHjd8acLOYUKRa5ueUgsekLHzPA2I)

## Tile Map

Data

![](https://api.monosnap.com/rpc/file/download?id=j70AczUj531TOgMMMc0lphliHcQlWZ)

Options

![](https://api.monosnap.com/rpc/file/download?id=hbtktvUgi1Yp5YzLmtfEbQMaCpvL67)

## Time Lion

https://github.com/elastic/timelion

설치: `~/lib/elk/kibana-4.6.2-darwin-x86_64 $ ./bin/kibana plugin --install elastic/timelion`

기간 설정

쿼리에 `.es(index="seoul-metro*", timefield="time_slot")`

`.es(index="seoul-metro*", timefield="time_slot", metric="sum:people_in"), .es(index="seoul-metro*", timefield="time_slot", metric="sum:people_out")`