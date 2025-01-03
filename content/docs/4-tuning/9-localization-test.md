---
title: "로컬라이제이션 테스트 (Localization Test)"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 408
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
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
rectangle "<b>직진 테스트</b>" as s5
rectangle "<b>경로 폭 조율</b>" as s6
rectangle "<b>회전 테스트</b>" as s7
rectangle "현재 단계" {
rectangle "<b>로컬라이제이션 테스트</b>" as s8
}
rectangle "<b>팔로워 PID 조율</b>" as s9

(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)
(s6) --> (s7)
(s7) --> (s8)
(s8) --> (s9)
```
현재 단계는 **로컬라이제이션 (2번째) 테스트** 단계입니다.

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
데드 휠을 사용하는 경우 이 페이지를 무시하세요.
{{< /callout >}}

## 로컬라이제이션 테스트

이 단계에서는 **드라이브 인코더를 사용한 로컬라이제이션이 정확한지** 확인합니다. 이 과정에서 발생하는 오류는 `DriveConstants` 파일의 문제 때문일 가능성이 높습니다.

1. RC를 통해 `LocalizationTest` 오프모드를 실행합니다.
2. RC에 연결된 스마트폰의 경우 `192.168.49.1:8080/dash`로, 컨트롤 허브(Control Hub)를 사용하는 경우 `192.168.43.1:8080/dash`로 접속합니다.
3. 대시보드 오른쪽 상단에서 **Field** 보기를 선택했는지 확인합니다.
4. 로봇을 이리저리 움직여 봅니다. 대시보드 맵에 로봇의 움직임이 그려져야 합니다.
5. 대시보드 상에서 표시된 로봇의 방향이 실제 로봇의 방향과 일치하는지 확인합니다.
6. 방향이 정확히 일치한다면 다음 단계로 진행합니다.  
   그렇지 않다면 이전 단계로 돌아가 오류를 수정해야 합니다.