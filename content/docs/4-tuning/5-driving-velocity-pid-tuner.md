---
title: "주행 속도 PID 조율 (Driving Velocity PID Tuner)"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 405
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

```kroki {type=PlantUML}

skinparam ranksep 30
skinparam dpi 125

rectangle "<b>데드 휠 설정</b>" as s2
rectangle "<b>로컬라이제이션 테스트</b>" as s3
rectangle "현재 단계" {
rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
}
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}
rectangle "<b>직진 테스트</b>" as s5

(s2) --> (s3)
(s3) --> (s4_y)
(s3) --> (s4_n)
(s4_y) --> (s5)
(s4_n) --> (s5)

```
현재 단계는 **주행 피드포워드 조율** 단계입니다.

이 내용은 추후에 추가합니다.