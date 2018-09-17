# LogAnalyzer_ELK
> ELK 활용 관리자 접근로그 수집/분석/시각화

## 프로젝트 기능

* 웹 서비스는 보통 관리자 페이지와 사용자 페이지로 나뉜다.
관리자 페이지에서는 주로 개인정보와 같은 중요한 정보를 다루므로, 접근 이력에 대한 로그를 남기도록 되어있다.
또한 일주일 단위로 로그 파일을 수집, 분석하여 검토보고서를 작성해야 하는데  [  -  ]

* 해당 프로젝트를 통해 기대 할 수 있는 효과는 업무 자동화를 통한 요구인력 감소에 있다.
로그파일을 불러오는 과정을 배치프로그램으로 작성하여 주기적으로 이루어 지도록 할 것이며
불러온 로그파일은 전처리과정을 거쳐 JSON 포맷으로 엘라스틱 서치에 저장된다.(CRUD)

* 또한 사용자는 웹 상에서 분석된 데이터를 기존 테이블 포맷 혹은 대시보드 형태로 조회할 수 있으며 
사용자가 직접 조건을 설정하여 원하는 데이터를 조회 할 수 있다.

## 기대효과
* 관리자웹 접근이력 검토서 작성 자동화를 통한 업무 효율성 향상


## 개발환경
```
Java jdk1.8.0_181
WebServer apache 2.4.34
WAS tomcat 8.5.27
Framework MVC 기반 Spring / Spring Boot 2.0.4
Beats filebeat 6.4.0
Logstash logstash 6.4.0
Elasticsearch elasticsearch-6.4.0
Kibana kibana-6.4.0
IDE Spring Tool Suite 3.9.5 / Eclipse Photon (4.8.0)
```

## 프로젝트 구조

<img src="https://github.com/twooopark/LogAnalyzerELK/blob/master/image/struc.PNG" width="700px" height="300px" />

## 주어진 데이터
원본 데이터 예시
```
dloer_casadmin_access_18080811.log
[18/08/08 11:52:35] 123.45.25.201 | s12er798 | abcd/common/loginAct | Y | 
[18/08/09 17:11:08] 123.45.25.200 | jisdfun17 | abcd/user/userList | userList |
```
ElasticSearch Mapping 초안
```
"mappings": { # 타입(type) 정의
		"logfile": { # 타입 이름
			"properties" : { # 문서(document) 정의
				"conn_date": {"type": "date", “format”:”yyyy-MM-dd HH:mm:ss”} 	#접속 날짜
				,"conn_ip": {"type": "keyword"}						#접속 ip
				,"conn_id": {"type": "keyword"}						#접속 ID
				,"url": {"type": "keyword"}							#URL
				,"action": {"type": "keyword"}						#ACTION
				,"login_isSuccess": {"type": "keyword"}					#로그인 성공 여부
				,"detaile_id": {"type": "keyword"}						#조회 대상 ID
    }
	}
}
```
## 데이터 수집

* 여러 서버들에서 로그를 수집하는데에 유용한 오픈소스 Beats

* Beats에서 로그 수집에 특화되어있는 Filebeat

* Filebeat는 common log format(nginx, apache 등)을 각각에 해당하는 처리 모듈을 통해 손쉽게 파싱하고, ElasticSearch에 전달할 수 있다. 
→ 지원되는 common log format인지 확인 필요
EX) Filebeat의 nginx 포맷 접근 로그 수집 명령어
PS > .\filebeat.exe -e -M "nginx.access.var.paths=[c:/programdata/nginx/logs/*access.log*]"
* 우리는 접근 로그가 한 파일이 아니라 시간별, 서비스별로 로그 파일 분리되어 있다. 
→ paths: - /home/...경로.../*.log 처럼 경로 설정 부분에서 처리
(inode를 통해, 새로 생성된 파일에 대한 부분만 처리 할 수 있다.)
출처 : http://yongho1037.tistory.com/709
만일,  Common Format이 아니라면, Logstash를 통해 파싱하고, ES에 전달해야 할 것이다.
+) 간단한 파싱이라면, ElasticSearch의 수집부에서도 처리가 가능하기도 하다.

* 로그 포맷을 파악하고, Filebeat에서 파싱이 가능하다 판단되면, LogStash를 생략하고 ElasticSearch로 로그 데이터를 전달해도 된다.
파싱할 내용이 간단하다면, ElasticSearch에서 데이터를 수신하고 사용하기 이전에 파싱할 수 있다.

* 위 두 사항들을 충족시키는지 확인 후 LogStash를 사용할지 여부를 판단할 것이다.
* LogStash로 수집하지 않고 Filebeat를 사용하는 이유는?
→ Filebeat는 LogStash보다 시스템 공간과 사용 공간이 적다. 즉, LogStash를 사용하는 경우가 리소스를 더 소비한다. 하지만 LogStash는 다양한 소스에서 데이터를 수집할 수 있고 다양한 플러그인을 제공.출처: http://yongho1037.tistory.com/709 [게임 서버 개발 노트]


## 진행 과정
1. ELK Stack 설치
(ElasticSearch - kibana - Logstash - Filebeat) 순으로 설치 & 테스트 한다.
수집부 담당이라고 Logstash, filebeat만 하다가 엉망진창헛고생ㅎㅎ;
2. 담당 기능 Beats(수집**,가공), Logstash(수집,가공**), ElasticSearch(저장,처리), Kibana(분석, 시각화)
3. 데이터 흐름 : B - L - E - K

## 수집부 주차별 상세 계획
* 2주차 Log data 전처리(LogStash)
Log Format을 분석하여, LogStash를 생략할지 여부 파악.
(Log Format이 일반적인 포맷(apache, nginx에서 제공하는 접근 로그 포맷)이라면, LogStash 생략 가능,
또한, 전처리 과정이 간단하다면, ElasticSearch에서 전처리 가능)
로그 데이터의 결측값, 이상값 여부 파악하여 전처리
ElasticSearch와 HTTP통신을 통해 Json 형태로 가공된 데이터 전달.

* 3주차 Log data 수집 테스트(Filebeat)
로컬 PC를 서비스 서버로 가정하여 테스트
Filebeat → ElasticSearch 연동
Filebeat → Logstash 연동
두 파이프라인 비교 분석


* 4주차 Log data 수집 & 자동화
실제 서비스 서버로부터  log data 수집
시간별 데이터 수집&가공 자동화(배치 처리)

* 향후 기대 효과
사용자 접근 로그 뿐만 아니라 다른 형식의 데이터를 수집하는데 용이한 오픈소스이므로 다방면으로 활용이 가능하다.
또한 분산시스템 처리가 가능하다.


## 수집부 테스트 진행 단계
1. ELK Stack 설치
2. ElasticSearch - Kibana 연동
3. Filebeat(input(logfile)) - Elastic Cloud 연동 테스트 
4. Logstash(input(stdin)) - ElasticSearch - Kibana 스택 가능성 확인(DaouStockLineChart) 
5. Logstash(input(실제logfile가공)) - ElasticSearch - Kibana
6. filebeat Logstash(input(실제logfile가공)) - ElasticSearch - Kibana
7. filebeat(input(실제logfile가공)) - ElasticSearch - Kibana
8. 6번과 7번 과정 장단점 비교 분석
9. 실제 서비스 개발서버에 Filebeat설치 후, 


## Logstash
> 로그스태시는 다양한 형식의 로그데이터를 수집, 가공하여 다양한 형식으로 전달할 수 있도록 하는 오픈소스이다.
Input, Filter, Output으로 3개의 플러그인으로 사용한다.


1. 설치 및 테스트
2. 로그 파싱 (Filter-grok,mutate,date 등)
3. ElasticSearch로 전송

## Filebeat
> Filebeat는 2가지 주요 요소가 있다.
Inputs 과 Harvesters 이 두가지는 같이 일한다, 파일 끝을 잡고, output으로 데이터를 보낸다.
1. Harvester은 단일 파일을 읽는 것에 책임이 있다.하나의 파일엔 하나의 Harvester가 할당된다.
2. Input은 Haverter들을 관리하고, 주어진 경로로부터 찾고자하는 타입의 모든 파일을 찾아낸다.
정리하자면, Input을 통해, 주어진 경로의 모든 파일들을 찾고, 각각의 파일들을 Haverster에 연결한다.
+)ELK STACK 운영 시에, 오래된 데이터를 삭제하지 않아서,
디스크 공간문제가 생길 수 있는데, 이런 문제는 Curator를 사용하여 
손쉽게 데이터 보존기간과 데이터 최대 사용량을 설정하여 ELK의 디스크 공간 문제를 해결한다.

1. 설치 및 테스트
<img src="https://github.com/twooopark/LogAnalyzerELK/blob/master/image/filebeat/filebeat_window_install.png" width="670px" height="360px" />
2. 로그 수집 & ogstash로 전송 테스트
filbeat.yml 수정 후, ./filebeat -e -d "publish"



## 명령어 정리

* ./filebeat -e -c filebeat.yml -d "publish"
	1. -e : log to stderr and disable syslog/file output
	2. -c : configuration file 선택 (default : filebeat.yml)
	3. filebeat.yml : 설정 파일
	4. -d string : 디버그모드로 수행(offset 정보를 직접 확인 가능하도록 offset 정보를 registry파일에 저장한다.)
	5. -d "publish" : 특정 디버그 선택기(여기선 "publish")를 사용하도록 설정합니다.
	- 참고) https://www.elastic.co/guide/en/beats/filebeat/current/command-line-options.html#command-line-options
	
* bin/logstash -f logstash.conf --config.reload.automatic
	1. 자동 구성 재로드가 가능하므로 구성 파일을 수정할 때마다 로그 스레시를 중지했다가 다시 시작하지 않아도 됩니다.
	- 참고) https://www.elastic.co/guide/en/logstash/current/running-logstash-command-line.html
	
## Filebeat(input(logfile)) - Logstash - Elastic Cloud 연동 테스트 

1. Filebeat.yml
> logstash-tutorial.log를 읽어들여, Logstash로 보냅니다. (파이프라인)
```yml
filebeat.prospectors:
- type: log
  paths:
    - C:\ElasticStack\logstash-tutorial.log
output.logstash:
  hosts: ["localhost:5044"]
```

2. first_pipeline.conf
> input{} 플러그인을 통해, Filebeat(5044포트)로 데이터를 수신하고, fileter{} 플러그인에서 grok, geoip 플러그인을 통해, Apache서버 로그형식을 파싱하고, 파싱된 정보 중에서, IP주소 정보를 Geo Information 형태로 변환했습니다.

> Cloud와 연동하는 과정에서 시간이 많이 소요됐습니다. logstash.yml에 cloud.id와 cloud.auth 정보를 입력하라는 글들이 많았는데, 그렇게 하면, 커넥션 에러가 발생하고, 아래와 같이 -.conf 파일에서 계정 정보를 입력해주니 됐습니다. yml 파일을 수정했으면, 적용시키는 과정이 필요한 것 같은데, 저는 그 적용하는 과정을 생략해서 안됐을 수도 있습니다. 

```conf
input {
# filebeat Port : 5044
    beats {
        port => "5044" 
    }
}
 filter {
# Grok Filter Plugin for Parsing Web Logs(ApacheLog...)
    grok {
        match => {"message" => "%{COMBINEDAPACHELOG}"} 
    }
# the geoip plugin looks up IP addresses, derives geographic location information from the addresses, and adds that location information to the logs.
    geoip {
        source => "clientip"
    }
}
output {
  elasticsearch {
    hosts => "https://주소.ap-northeast-1.aws.found.io:포트"
    user => "elastic"
    password => "비밀번호"
        }
}
```


3. ElasticSearch에 전송되어서 Index가 생성되었는지 확인
> https://주소.ap-northeast-1.aws.found.io:포트/_cat/indices?v

4. 실제로 전송된 데이터 확인, fileter 플러그인을 통해 message 부분이 파싱된 것을 확인 할 수 있다.
> https://주소.ap-northeast-1.aws.found.io:포트/logstash-2018.09.03/_search?pretty&q=response=200
<img src="https://github.com/twooopark/LogAnalyzerELK/blob/master/image/filebeat-logstash-elasticCloud2.PNG" width="700px" height="1000px" />

## Logstash / grok-pattern
- grok 정규식 패턴 : https://github.com/logstash-plugins/logstash-patterns-core/blob/master/patterns/grok-patterns
- 정규식 테스트 : http://grokconstructor.appspot.com/do/match
- 정규식 참고(http://ccambo.blogspot.com/2014/10/regular-expression.html)
- 문제점 : 데이터의 타입을 모두 String으로 출력한다. timestamp는 date 타입으로 하고 싶으나, 플러그인 상에서 바꿀 수 없다. 이 문제는 ElasticSearch의 동적 인덱스 생성기법을 사용하지 않고, 매핑을 미리 만들어둔 후, logstash에서 데이터를 전송해서 해결해보겠다.
데이터 타입 문제 관련 내용(https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#_grok_basics)
```
filter {
 grok {
  match => {
   "message" => 
   [		
    "\[(?<timestamp>%{YEAR}/%{MONTHNUM}/%{MONTHDAY} %{TIME})\] %{IPORHOST:clientip} \| %{USER:adminid} \| (?<path>%{WORD}%{UNIXPATH}) \| %{WORD:action} \| %{USER:userid}?"
   ]
  } 
 }
}
```
- 원본 데이터 예시 : 
```
[18/08/08 11:52:35] 123.21.25.206 | 1dfa213 | n/common/loginAct | Y | 

...

[18/08/09 16:41:10] 123.21.25.200 | 1id1df13 | n/user/userDetail | userDetail | 1id1213
```
- 결과 : 
```
[18/08/08 11:52:35] 123.21.25.206 | 1dfa213 | n/common/loginAct | Y |
MATCHED
adminid	1dfa213
path	/common/loginAct
timestamp	18/08/08·11:52:35
userid	
clientip	123.21.25.206
func	n
action	Y

...

[18/08/09 16:41:10] 123.21.25.200 | 1id1df13 | n/user/userDetail | userDetail | 1id1213
MATCHED
adminid	1id1df13
path	/user/userDetail
timestamp	18/08/09·16:41:10
userid	1id1213
clientip	123.21.25.200
func	n
action	userDetail
```


## Logstash -> ElasticSearch 매핑 문제
> Logstash filter grok 플러그인을 사용하는데, 결과물 타입이 모두 String타입으로 출력됩니다. 이것은 필터 상에서 해결되지 않으며,
ElasticSearch에 미리 매핑을 만들어 두어야 합니다. 여기서 매핑은 도큐먼트 처리 방법을 정의한 것으로 SQL의 테이블 정의 개념과 유사합니다.

> Es에서는 매핑을 만들지 않으면, 스스로 기본 매핑을 생성합니다. (이 기본 매핑이 우리 데이터 형식에 맞게 매핑되지 않고 있음)

> 해결방안 : ElasticSearch의 인덱스 탬플릿 생성
es에서 put방식으로 넣으면 됨.
https://github.com/twooopark/LogAnalyzerELK/blob/master/_template-elog_index_template.txt

> 또 다른 문제, 날짜 형식 변환 (target 때문에 )
```json
date {
	match => ["conn_date", "YY/MM/DD HH:mm:ss"]   					
	target=> ["conn_date"]
	timezone => "UTC"
	locale => "ko"
}
```

## filebeat는 어떻게 로그파일을 불러오는가?
* registry 파일, 로그파일에 내용 추가했을때, offset과 timestamp도 같이 바뀜, 
```json
{
"source":"C:\\Program Files\\Filebeat\\AdminLog\\flora_test_access_18071715.log",	//로그파일경로
"offset":468,		//tail 위치
"timestamp":"2018-09-04T17:35:16.9678126+09:00", 	// 마지막으로 읽은 시점
"ttl":-1,
"type":"log",
"meta":null,
"FileStateOS":
	{
	"idxhi":1966080,
	"idxlo":347009,
	"vol":4272810600
	}
},
...
```


## logstash -> elasticSearch 보내는 과정에서 오류
- conn_date의 월을 나타내는 부분이 자꾸 date filter에서 모두 1월로 바뀌는 현상
- match 할때, YY/MM/DD로 했는데, DD는 Day of year, dd는 Day of month를 나타낸다고 함.
- 참고) https://stackoverflow.com/questions/21988900/wrong-month-number-when-parsing-datetime-string-with-date-filter
- 수정 결과
``` json
date {
	match => ["conn_date", "yy/MM/dd HH:mm:ss"]   					
	target=> ["conn_date"]
	timezone => "UTC"
	}
}
```



## ElasticSearch
- 인덱스(색인)
```
인덱스는 비슷한 형질을 가지는 문서 간의 집합이라고 보시면 됩니다. 
예로들면 고객정보, 제품카탈로그, 주문정보 같은 것 입니다. 
소문자로 구성된 이름으로 구분이 되며 인덱스 이름은 문서에 대한 인덱싱/검색/갱신/삭제 등을 수행할 때 참조값으로 사용됩니다.
단일 클러스터에서 원하는 대로 다수의 인덱스를 정의할 수 있습니다.
```
- 문서
```
문서는 인덱싱된 정보의 기본 단위입니다. 예로 하나의 고객 정보, 제품 정보, 주문 정보 입니다. 
문서는 어디든 호환이 가능한 JSON(JavaScript Object Notation)으로 구성됩니다.

인덱스/타입과 함께 원하는 만큼의 문서를 저장할 수 있습니다. 
문서는 물리적으로 인덱스에 있기는 하지만, 필히 인덱스 안에 있는 타입을 기준으로 인덱싱/할당 되어야 합니다.
```
- 매핑(Mapping과 Template)
```
Elasticsearch가 인덱스에 자료를 저장할 때, 그것을 어떻게 다룰 것이며, 
어떤 자료형으로 처리할 것인지 등을 정의하는 것을 Mapping이라고 한다. 

이러한 Mapping 정의와 인덱스 설정 등을 담아 두었다가 
인덱스가 생성될 때 자동으로 참고할 수 있도록 해주는 것이 Template이다.
```
- 인덱스 탬플릿
```
인덱스 템플릿은 새로운 인덱스를 생성할 때 적용될 내용을 미리 템플릿으로 만들 수 있게 해줍니다. 
템플릿은 설정과 매핑 정보를 포함할 수 있으며, 템플릿 패턴을 my_index*로 하면 my_index* 패턴에 
일치하는 이름의 인덱스를 생성할 경우 템플릿의 설정과 매핑정보를 따르게 됩니다.
```
- ElasticSearch.yml 적용 문제
	- .\elasticsearch -Enetwork.host=0.0.0.0 -Enetwork.port=9200와 같이 -E 뒤에 설정 명령어를 추가해서 해결. 
	- 원래는 .\elasticsearch 실행하면 elasticsearch.yml이 적용되나본데... 왜 안되는진 모르겠음.



## 새로운 로그파일 format을 처리 하는데 생긴 문제점 (데이터 포맷, encoding 방식)
> 여러 형태의 데이터를 한번에 받아서, 각각 동일한 형태로 파싱(가공)하여, ElasticSearch로 전달해야 한다.
1. 새로운 입력 포맷도 파싱하기 위해 여러 시도를 했다.
	a. multiple filter : filter를 중복해서 만들 수 없다고 함
	b. multiple pipeline : logstash를 여러개 운영하는 방식 > 메모리 소모가 클 것으로 예상되서 배제
	c. multiple grok : filter의 플러그인 중 하나인 grok은 데이터 파싱하는데 유용한 플러그인 인데, 조건문에 따라 다른 grok 플러그인을 실행하도록 한다. -> grok 특성 상, input이 들어오면 모두 거치게 되는데, 이때 수많은 error가 발생할 것이다.
	d. 단일 grok에서 message를 다양한 형식으로 처리 할 수 있음. >> 파싱 해결
	
2. encoding 방식, 새로운 로그 포맷은 한글이 포함되어 있음
> default로 utf-8 방식으로 작동하는 ELK, 한글이 포함된 로그를 분석하기위해 euc-kr 방식으로 변환해야 했다.
	a. logstash의 input부분에서 plain 플러그인을 통해 charset을 eucKR, CP949 등으로 바꿔보았지만, 占쏙옙회 같은 문자가 나옴.("조회")가 출력되어야 한다.
	b. 앞 글자가 깨지는 현상의 원인을 파악해보니 utf-8은 3byte, euc-kr은 2byte로 문자를 인식하고, 서로 변환하는 과정에서 데이터가 손실될 가능성이 있다고 생각됨.
	c. euckr/utf-8.log > filebeat > logstash
	
	
	
## Filebeat > logstash > elasticSearch > kibana 동작 과정 및 
> 3대의 서버를 사용했고, 각각의 서버에 filebeat를 하여 로그를 수집하도록 했습니다. 
서버1(Logstash, Filebeat), 서버2(ElasticSearch, Filebeat), 서버3(Kibana, Filebeat) 로 설치합니다.
1. 3대의 서버의 Filebeat로 서버1의 logstash에 로그 파일을 json형식으로 http를 통해 보냅니다. (filebeat, input : logfile, output : 서버1)
2. Logstash는 input : beats로 받고, grok을 통해 각기 다른 로그형식을 통일화시켜 서버2의 es로 보냅니다.(elasticSearch, intput : beats, output : 서버2)
3. elasticsearch를 localhost가 아닌 다른 ip에서 접속하기 위해 network Setting을 해줍니다. (.\elasticsearch -Enetwork.host=0.0.0.0)
4. 서버3의 kibana는 서버2의 es에 접속하여, 정형화된 로그데이터들을 시각화, 분석합니다. (kibana.yml, elasticsearch.url: "서버2:9200")
5. kibana를 localhost가 아닌 다른 ip에서 접속하기 위해 network Setting을 해줍니다. (kibana.yml, server.host: "0.0.0.0")



## 로그 데이터 정형화를 위한 데이터 필터 API 아이디어
```
1. 로그 데이터의 아이템 항목 갯수만큼 입력 칸 추가/삭제
EX) "[날짜] IP ID" 인 경우, 3개를 추가해야한다.
ㅁ[dropdown]ㅁ[dropdown]ㅁ[dropdown]
ㅁ : 은 구분자. 맨 마지막은 공백이 아닌 엔터로 인식.
아무것도 경우 공백1칸으로 인식(X, 이렇게 하면 안될듯. 로그 뒷 부분이 구분자 없이 그냥 끝날 경우 에러가 생길 것임)
dropdown : 날짜, IP, ID, Action, URI, Detail이 드롭다운으로 나타남. (mapping에 지정되어있는 항목만 나타냄) -> 항목 중복선택 불가, 항목 추가하기 기능 필요(매핑은 재설정이 불가함 >> 매핑추가(?))

2. 이미 만들어진 데이터 필터를 보여줘야함.
1번과 같이 필터를 이미 생성한 경우, 서비스명 : "[날짜] IP ID" 과 같이 이미 있는 필터를 보여줌.
그리고 각 데이터 필터가 적용되고 있는 서비스를 보여주며, (한 서비스에서 다른 로그 데이터 형식을 출력할 수 있으므로 서비스별 필터 여러개 가능)
중복으로 필터를 추가하려는 노력을 방지함. (사용자를 위한 것임. 중복된 필터 추가 차단 기능은 구현되어있어야함)

3. 생성된 필터는 정규식으로 적절하게 변환, grok 또는 dissect 플러그인에 적용, 파일 수정, logstash는 reload.automatic으로 하여, 재시작 필요 없도록 함. (테스트 필요)

4. 향후 게획 

일정량 이상의 데이터를 한 줄 단위로 
[18년2월1일] 123.123.12.33 abc마트
[19년6월1일] 123.123.22.31 a101트
[18년1월1일] 123.123.52.33 a마녀랑 와 같이 입력하면, 
(1) 자동으로 이미 필터가 존재하는지 판단해주는 기능.
--> 선형탐색으로 처리 가능.

(2) 자동으로 필터를 예측해주는 기능. 1번에서 하는 일을 대신 해줌으로써, 사용자의 수고를 덜을 수 있음.
--> 소요기술 : 반복되는 패턴에 대한 감지(패턴인식 알고리즘) 

(3) 한 줄의 데이터로 엔터만 쳐서, 간편하게 필터 만들기
[
날짜
] 
IP
 
ID
처럼 구분자와 데이터 항목을 분류해서...? (공백 문자로 인해 실수하기 쉽기 때문에, 안 좋은 듯)
--> 뭐가 날짜인지 IP인지 ID인지 스스로 파악할 수 있는 알고리즘 필요... 제일 별로인 아이디어...
```

## 참고자료 regex Generators

- [Txt2Re](http://www.txt2re.com/index.php3) - Generate Regular expressions based on a string
- [Regex Generator++](http://regex.inginf.units.it) - Automatic Generation of Text Extraction Patterns from Examples
- [regexgen](https://github.com/devongovett/regexgen) - Generates regular expressions that match a set of strings.
- [RegexGenerator](https://github.com/MaLeLabTs/RegexGenerator) - A tool for generating regular expressions for text extraction (by @MaLeLabTs)
- [Gamon's numberic range generator](http://gamon.webfactional.com/regexnumericrangegenerator/) - Regex Numeric Range Generator, when you need to match an integer range.
- [rgxg](https://rgxg.github.io) - Command line tool to generate Regex


## 필터 생성기 초안
<img src="https://github.com/twooopark/LogAnalyzerELK/blob/master/image/filtergenerator.PNG" width="800px" height="350px" />

```
파일명과 로그데이터를 입력하고, 입력 버튼을 누르면, 
지정된 구분자에 따라 문자들이 나눠진다.
나눠진 문자들은 우측과 같은 각 항목 별로 drag & drop을 이용해 삽입한다.
문자(item)들은 각각 span 태그로, id는 순서대로 drag1 ... drag번호 로 생성된다.
각 구분자(sep)도 순서대로 번호를 매김으로써 필터 생성 시 dragN과 구분자를 매칭시켜 만들 수 있도록 한다.

IP와 시간은 사용자의 편의성을 위해 합쳐져 있도록 추가적인 처리가 필요하다.
```


# 필터 생성기 동작방식(알고리즘) 초안
```
위의 filename을 예시로 들면,
1. 가장 좌측부터 문자 하나씩 읽어나간다. 
2. 구분자(sep)가 있는지 확인한다.
	2-1. O : sep[0]에 sep을 담는다.
	2-2. X : sep[0]에 아무것도 담지 않는다.
3. 코드 작성하면서 생각하자...
```
```
만들어야 할 필터 예시...
filename >> %{SEP:server}\_%{SEP}\-%{SEP}\.%{SEP:service}\.%{SEP}\_%{SEP}\_%{SEP:day1}\.%{SEP:filetype}
cf) SEP = /[^\-\_\^\.\s\[\]\|\:]+/;

1. 구분자에 따라 \X%{WORD2}를 item 수 만큼 String 배열로 만든다.
2. 첫 구분자 유무에 따라 첫 배열의 \X를 제거한다.
3. 그리고 필드에 추가된 item은 배열에서 찾아 %{WORD2}에서 %{WORD2:필드명}으로 바꾼다.
	Date >> %{DATENUM2:access_date}
	IP >> %{IPORHOST:access_ip}
	id >> %{USER:access_id}
	uri, action, remark>> %{DATA:~}
4. 가장 마지막에 "$"을 추가해야, 마지막에 있는 값이 제대로 들어간다.	 

개선안 >>
정규식으로 구분자, 구분자 아님을 구분만 하도록 변경함.
noSep [^\-\_\^\.\s\[\]\|\:]+
Sep [\-\_\^\.\s\[\]\|\:]+
%{Sep}?%{noSep:server}%{Sep}%{noSep}%{Sep}%{noSep}%{Sep}%{noSep:service}%{Sep}%{noSep}%{Sep}%{noSep}%{Sep}%{noSep:day1}%{Sep}%{noSep:filetype}$
맨 처음엔 구분자 없을수도 있어서 ?로 함.
		
아직 날짜는 어떻게 처리할지 모르겠음. ㅎㅎ; 어렵다... 사용자의 추가적인 입력(날짜형식선택)이 필요할 수도
1. items의 갯수 파악(n개). 양쪽에 구분자 없는 경우, n-1개로 처리해야함
2. n개 만큼 %{Sep}%{noSep}을 만들 건데, 그 중 필드데이터가 뭔지 알아야한다.
3. 아래와 같이 각 필드에 해당하는 데이터를 읽을 수 있다.
	for (fn in fieldName)
		fieldData = document.getElementById(fieldName[fn]).innerText;
4. 하지만, 내가 필요 한 건, 위에서 찾은 div에 하위 항목인 span의 id가 필요하다.
filenameItem3이면, 서버명은 파일네임 아이템들 중에  0123 4번째에 있는 것이다.
>> document.getElementById(fieldName[fn]).children[0].id
5. ...
```
