---
layout: single
title: '[ElasticSearch] ElasticSearch 클러스터링'
categories: ElasticSearch
tag: [Elastic Search]
toc: true 
author_profile: false
sidebar:
    nav: "counts"
published: false

---


<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-09-06-elastic-search-cluster\es_cluster.png" alt="Alt text" style="width: 80%; height: 80%; margin: 20px">
</div>



## ElasticSearch 노드 

Elasticsearch는 **여러 대의 노드들이 각자의 역할을 기반으로 서로 연결되어, 마치 하나의 통합된 시스템처럼 동작하도록 설계**되어 있다. 각 노드는 특정 역할을 담당하며, 이들 노드가 협력하여 데이터 저장, 검색, 관리 등의 기능을 수행한다. 이를 통해 분산된 환경에서도 안정적이고 효율적인 데이터 처리가 가능해진다.

<table>
  <thead>
    <tr>
      <th style="font-size: 16px;">종류</th>
      <th style="font-size: 16px;">역할</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="font-size: 16px;">마스터 노드</td>
      <td style="font-size: 16px;">클러스터 상태 관리 및 메타데이터 관리</td>
    </tr>
        <tr>
      <td style="font-size: 16px;">데이터 노드</td>
      <td style="font-size: 16px;">문서 색인 및 검색 요청 처리</td>
    </tr>
        <tr>
      <td style="font-size: 16px;">코디네이팅 노드</td>
      <td style="font-size: 16px;">검색 요청 처리</td>
    </tr>
        <tr>
      <td style="font-size: 16px;">인제스트 노드</td>
      <td style="font-size: 16px;">색인되는 문서의 데이터 전처리</td>
    </tr>
</tbody>
</table>


**마스터 노드**는 클러스터 차원의 작업을 관리하는 역할을 수행한다. 예를 들어, 인덱스 생성 및 삭제, 클러스터 내 노드 상태 감시, 노드별로 할당될 샤드 결정 등 클러스터 내에서 일종의 **관리자 역할**을 한다

마스터 노드에 문제가 발생할 경우를 대비해, **마스터 후보 노드(master-eligible)**들이 존재할 수 있다. 기본적으로 마스터 후보 노드는 마스터 노드가 관리하는 클러스터 메타 데이터를 "공유" 받기 때문에, 마스터 노드에 장애가 발생할 시 즉각 대체가 가능한 구조이다. 이 후보 노드들은 실제 마스터 역할을 수행하지 않기 때문에, 실제 마스터 노드보다 적은 heap과 CPU 스펙을 가질 수 있으며, 다른 노드 역할도 수행할 수 있다.


코디네이트 노드는 데이터를 저장하거나 사용자의 요청을 직접 처리하지 않지만, 요청을 전달하고 결과를 취합하는 역할을 한다. 일종의 로드밸런서로 생각할 수 있다.



## 인덱스와 샤드

인덱스는 **문서가 저장되는 논리적인 공간**이다. 인덱스는 사용 패턴과 문서의 특성에 따라 설계해야 한다. 


<table>
  <thead>
    <tr>
      <th style="font-size: 16px;">케이스</th>
      <th style="font-size: 16px;">장점</th>
      <th style="font-size: 16px;">단점</th>
    </tr>
  </thead>
  <tbody>
     <tr>
      <td style="font-size: 16px;">하나의 인덱스를 사용할 때</td>
      <td style="font-size: 16px;">관리해야 할 인덱스의 수가 적어 관리 리소스가 적게 발생한다</td>
      <td style="font-size: 16px;">쿼리와 문서의 구조가 복잡해 질 수 있다.</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">여러개의 인데스로 나눌 때</td>
      <td style="font-size: 16px;">각각의 경우에 최적화된 쿼리와 문서 구조를 사용할 수 있다.</td>
      <td style="font-size: 16px;">관리해야 할 인덱스의 수가 많아 관리 리소스를 발생할 수 있다.</td>
    </tr>
</tbody>
</table>

처음에는 하나의 인덱스로 단순하게 시작하는 것이 좋다. 이후 사용 패턴에 따라 인덱스를 별도로 운영하는 방식으로 확장하는 것을 추천한다.


## 샤드 (shard)

<div style="display: flex; justify-content: center;">
     <img src="{{site.url}}\images\2024-09-06-elastic-search-cluster\shard.png" alt="Alt text" style="width: 80%; height: 80%; margin: 20px">
</div>


샤드는 **인덱스에 색인된 문서가 실제로 저장되는 물리적인 공간**을 의미한다. 하나의 인덱스는 반드시 하나 이상의 샤드를 가지며, 이를 통해 데이터를 분산 저장하고 처리할 수 있다.


<table>
  <thead>
    <tr> 
      <th style="font-size: 16px;">종류</th>
      <th style="font-size: 16px;">역할</th>
    </tr>
  </thead>
  <tbody>
     <tr>
      <td style="font-size: 16px;">프라이머리 샤드</td>
      <td style="font-size: 16px;">문서가 저장되는 원본 샤드, 색인과 검색 성능에 모두 영향을 줌</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">레플리카 샤드</td>
      <td style="font-size: 16px;">프라이머리 샤드의 복제 샤드, 검색 성능에 영향을 줌<br>프라이머리 샤드에 문제가 생기면 레플리카 샤드가 프라이머리 샤드로 승격</td>
    </tr>
</tbody>
</table>


문서가 샤드에 어떻게 저장되는지를 결정하는 과정을 **샤드 라우팅**이라고 하며, 기본적으로 **문서ID % 샤드의 개수**라는 라우팅 규칙을 사용한다. 이 규칙에 따라 문서들이 샤드에 고르게 분배된다. 그러나 샤드의 개수가 변경되면 문서가 저장되는 규칙이 완전히 바뀌게 된다.

또한, 인덱스 템플릿을 통해 인덱스 생성 시 샤드의 개수를 미리 설정할 수 있다. 이는 인덱스 설계 시 중요한 고려사항이다.


- 인덱스는 문서가 저장되는 논리적인 공간이며, 샤드는 인덱스에 색인된 문서를 실제로 저장하는 물리적인 공간
- 인덱스 설계는 Elasticsearch를 사용할 때 가장 먼저 고려해야 할 중요한 요소
- 프라이머리 샤드는 문서가 저장되는 원본 샤드이며, 레플리카 샤드는 프라이머리 샤드의 복제본으로, 데이터의 가용성과 안정성을 높이는 역할
- 문서가 샤드에 라우팅되는 규칙은 (문서ID) % (샤드의 개수)에 따라 결정되므로, 인덱스 생성 이후에는 프라이머리 샤드의 개수를 변경할 수 없다
- 인덱스 템플릿을 통해 인덱스 생성 시 샤드의 개수를 미리 설정할 수 있다. 


## 매핑(mapping)
핑은 Elasticsearch에 저장되는 문서들의 구조를 나타내는 정보이다. 흔히 Elasticsearch가 스키마리스(schema-less)라고 오해되지만, 실제로는 스키마를 미리 정의하지 않아도 될 뿐이지, 문서의 구조는 중요하다.

<table>
  <thead>
    <tr> 
      <th style="font-size: 16px;">종류</th>
      <th style="font-size: 16px;">정의</th>
    </tr>
  </thead>
  <tbody>
     <tr>
      <td style="font-size: 16px;">동적 매핑(Dynamic Mapping)</td>
      <td style="font-size: 16px;">처음 색인되는 문서를 바탕으로 매핑 정보를 ElasticSearch가 동적으로 생성 </td>
    </tr>
     <tr>
      <td style="font-size: 16px;">정적 매핑(Static Mapping)</td>
      <td style="font-size: 16px;">문서의 매핑 정보를 미리 정의</td>
    </tr>
</tbody>
</table>

### 정적 매핑이 필요한 경우
- 문서 필드의 타입 지정: 문서의 필드가 가지는 값에 따라 타입을 명확하게 지정해야 할 때, 정적 매핑이 필요하다. 예를 들어, float 타입의 최대 값을 넘는 값이 들어갈 가능성이 있다면, 반드시 double 타입으로 정적 매핑을 설정해야 한다.

- 불필요한 색인을 방지하기 위해: text 필드의 경우, 기본적으로 색인되며, 검색 가능한 상태로 저장됩니다. 이때, keyword 타입으로도 함께 매핑되기 떄문에 불필요한 색인이 발생 할 수 있다. 

## 색인 (Indexing)
색인은 문서를 분석하고 저장하는 과정을 의미한다. Elasticsearch에서 색인은 문서를 인덱스에 추가하는 중요한 과정으로, 데이터의 검색 가능성을 높이는 역할을 한다.

### 색인 (Indexing) 과정

색인은 인덱스를 생성하고, 매핑을 확인하며, inverted index를 생성하고 문서를 저장하는 일련의 과정을 포함한다.

색인은 주로 **프라이머리 샤드**에서 일어나며, 기본적으로 number_of_shards의 값은 1로 설정되어 있는데, 이 경우 클러스터로서의 이점을 충분히 활용하지 못할 수 있다. 적절한 샤드의 개수를 설정하는 것은 색인 성능을 향상시키는 데 중요한 역할을 한다.

색인 성능에 문제가 있다면, 샤드의 수를 늘리거나 데이터 노드를 스케일 아웃하면서 최적의 설정을 찾아가는 것이 필요하다. 

### 검색 과정

1) 검색어 분석: 먼저, 입력된 검색어에 애널라이저가 적용되어 분석된다.
2) inverted index 검색: 생성된 토큰을 기반으로 inverted index에서 검색이 수행된다.
3) 검색 결과 표시: 검색된 결과가 사용자에게 표시된다.

#### Inverted Index
 분석된 문자의 결과를 저장하는 데이터 구조체로, 검색의 효율성을 높이는 역할을 한다.

#### 애널라이저 (analyzer)

문자열은 다음 단계를 거쳐 분석된다
- Character Filter: 문자열의 특정 문자를 변환하거나 제거합니다.
- Tokenizer: 문자열을 토큰으로 분할합니다.
- Token Filter: 토큰을 추가로 변환하거나 필터링하여 최종 토큰을 생성합니다.

이 과정을 통해 생성된 토큰은 inverted index를 구성하는 데 사용된다.

검색 요청은 프라이머리 샤드와 레플리카 샤드 모두에서 처리될 수 있어, 데이터 가용성과 검색 성능을 높이는 데 기여한한다.



**Elasticsearch는 분산된 클러스터 환경에서 동작하기 때문에, 모든 노드가 색인과 검색 작업을 효율적으로 처리할 수 있도록 구성하는 것이 매우 중요하다.** 

### text 와 keyword 타입

- text 타입
     - 전문 검색(Full-Text Search)을 위해 사용된다. text 타입 필드는 공백을 기준으로 토큰을 생성하여 분석하고, 이를 통해 복잡한 검색 쿼리를 수행할 수 있다.

- keyword 타입
     - 정확한 일치(Exact Matching)를 위해 사용됩니다. keyword 타입 필드는 토큰 생성 없이 그대로 저장되며, 검색 시에도 토큰화를 거치지 않기 때문에 색인 속도가 더 빠르다.


문자열 필드가 동적 매핑되면, Elasticsearch는 자동으로 text와 keyword 두 가지 타입으로 필드를 생성한다.

문자열 필드의 특성에 따라 text와 keyword를 정적으로 매핑해주는 것이 성능에 도움이 된다. 예를 들어, 토크나이징이 필요한 필드는 text 타입으로, 그렇지 않은 필드는 keyword 타입으로 설정하는 것이 좋다.

- text로 정의하면 좋은 필드 : **주소, 이름, 물품 상세정보 등**

- keyword로 정의하면 좋은 필드 : **성별, 물품 카테고리 등**


## CAT API
CAT API는 Compact and Aligned Text의 약자로, Elasticsearch 클러스터의 정보를 사람이 읽기 쉬운 형태로 출력하기 위한 API이다. 

- cat health : ElasticSearch 클러스터의 전반적인 상태를 확인할 수 있다.

  
<table>
  <thead>
    <tr> 
      <th style="font-size: 16px;">상태</th>
      <th style="font-size: 16px;">의미</th>
    </tr>
  </thead>
  <tbody>
     <tr>
      <td style="font-size: 16px;">green</td>
      <td style="font-size: 16px;">프라이머리 샤드, 레플리카 샤드 모두 정상적으로 각 노드에 배치되어 동작하고 있는 상태</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">yellow</td>
      <td style="font-size: 16px;">프라이머리 샤드는 정상적으로 동작하지만 일부 레플리카 샤드가 정상적으로 배치되지 않은 상태<br>색인 성능에는 이상 없지만 검색 기능에는 영향을 줄 수 있다.</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">red</td>
      <td style="font-size: 16px;">일부 프라이버머리 샤드와 레플리카 샤드가 정상적으로 배치되지 않은 상태<br>색인 성능, 검색 성능에 모두 영향을 주며 문서 유실이 발생할 수 있다.</td>
    </tr>
</tbody>
</table>

- cat nodes : 노드들의 전반적인 상태 확인 
노드들의 디스크 사용량 확인, 노드들이 명확한 역할을 수행하고 있는지 확인, 어떤 노드가 마스터 노드인지 확인, 노드들의 메모리 사용량 확인

- cat indices : 인덱스의 상태를 확인 할 수 있습니다. 
인덱스들의 프라이머리 샤드 개수와 레플리타 개수 확인, 이상 상태인 인덱스 확인

- car shards: 샤드의 상태 확인 
이상 상태인 샤드들 확인, 샤드들의 이상상태 원인 확인 

## ElasticSearch 인덱스 설계하기 

### 개발환경 

- 운영체제: Windows 11 Home
- Elasticsearch 버전: 8.15.0
- Docker 버전: 27.1.1
- Python 버전: 3.10

### 데이터
- 국토교통부아파트 실거래 데이터
- 형태 : csv 파일
- 컬럼 

<table>
  <thead>
    <tr> 
      <th style="font-size: 16px;">Field</th>
      <th style="font-size: 16px;">Data Type</th>
      <th style="font-size: 16px;">Description</th>
    </tr>
  </thead>
  <tbody>
     <tr>
      <td style="font-size: 16px;">district</td>
      <td style="font-size: 16px;">keyword</td>
      <td style="font-size: 16px;">시군구 : 서울특별시 구로구 신도림동</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">lotNumber</td>
      <td style="font-size: 16px;">text</td>
      <td style="font-size: 16px;">번지</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">mainNumber</td>
      <td style="font-size: 16px;">float</td>
      <td style="font-size: 16px;">본번</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">complexName</td>
      <td style="font-size: 16px;">text</td>
      <td style="font-size: 16px;">단지명</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">areaSqm</td>
      <td style="font-size: 16px;">float</td>
      <td style="font-size: 16px;">전용면적(㎡)</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">contractDay</td>
      <td style="font-size: 16px;">date</td>
      <td style="font-size: 16px;">계약일</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">transactionAmount</td>
      <td style="font-size: 16px;">int</td>
      <td style="font-size: 16px;">거래금액(만원)</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">building</td>
      <td style="font-size: 16px;">text</td>
      <td style="font-size: 16px;">동</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">floor</td>
      <td style="font-size: 16px;">keyword</td>
      <td style="font-size: 16px;">층</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">yearBuilt</td>
      <td style="font-size: 16px;">keyword</td>
      <td style="font-size: 16px;">건축년도</td>
    </tr>
     <tr>
      <td style="font-size: 16px;">roadName</td>
      <td style="font-size: 16px;">keyword</td>
      <td style="font-size: 16px;">도로명</td>
    </tr>
    <tr>
      <td style="font-size: 16px;">transactionType</td>
      <td style="font-size: 16px;">keyword</td>
      <td style="font-size: 16px;">거래유형</td>
    </tr>
    <tr>
      <td style="font-size: 16px;">brokerLocation</td>
      <td style="font-size: 16px;">text</td>
      <td style="font-size: 16px;">중개사소재지</td>
    </tr>
    <tr>
      <td style="font-size: 16px;">registrationDate</td>
      <td style="font-size: 16px;">date</td>
      <td style="font-size: 16px;">등기일자</td>
    </tr>
</tbody>
</table>


## 인덱스 템플릿 


현재는 아파트 실거래 매매가 데이터만 다룰 예정이므로  **하나의 인덱스**로 구성된 구조로 설계할 예정이다. 


<br>
<br>

----
Reference

- <a href = 'https://www.inflearn.com/course/elasticsearch-essential/dashboard'>ElasticSearch Essential</a>
- <a href = 'https://jeongxoo.tistory.com/12'>엘라스틱 서치의 구성 요소 및 구조</a>

