---
layout: single
title: '코멘토 SQL 입문부터 활용까지 마지막 피드백과 회고 '
categories: 코멘토
tag: [코멘토]
author_profile: false
published: false
sidebar:
    nav: "counts"
---

데시보드는 시점에 관한 명확한 기준이 있어야함 

분석보고서 - 주제에 대해 데이터를 여러 방향으로 살펴봄 (주로 과거의 데이터를 이용함)

대시보드 - 인사이트를 내기 위해서 사용 , 
큰틀을 위한 대시보드 - 매출 현환, 과거 대비 추이
업무를 위한 대시보드 - 마케팅 성과 대시보드

L

--- 피드백

전년도 대비 보다는 전연도 분기 대비로 하는것이 더 합리적 

total과 전연도 대비 

대시보드는 중요한 지표들을 보여주는 화면이기 때문에 한 차트에 많은 정보를 보여주기위해 노력하삼 

월별 매출 증감률 -> 전월 대비인건지, 연도별인건지 설명이 부족함 . 대시보드는 누구나 알아볼 수 있게 병확한 설명이 필요함 ! 만든 사람이 의도가 잘 나타내야함 


파이차트 - 비율을 표현 할떄에 잘함

파이차트는 절대적인 값보다 상대적인 비교에 추천 

---
상위 5개를 나타내되 나머지를 기타로 합쳐서 옵션으로 만들어줌! top5와 나머지 이런식으로 !!! 

연도별 비중이 거의 비슷하다면 굳이 주목해서 봐야할 지표는 아닐것같음.. 대시보드는 의미있는 지표만! 
대시보드는 인사이트를 볼 수 있는 지표만 나타내라! 대시보드에 담기기에 적절하지 않은 지표일 수 있다.

변동이 많은 지표를 찾아서 추가 ...

주문수를 볼떄는 사람수 기준으로 분석하는것이 좋다.

대시보드를 채울지보다는 어떻게 간추려서 담을지에 집중. 

보조선으로 평균 표시 


date_add(curdate(), interval -7 day)


좋은 가설 수립 - 좋은 질문을 던진다.. 데이터가 있는 DB가 있으면 꼬리를 물고 생각나는 의문들을 분석해나가는 과정을 하다보면 어느순간 특이 데이터를 알 수가있음 

나를 위한 대시보드를 한번 만들어봐랑! 중요한 지표들을 대시보드화해서 모아놓아라! 
