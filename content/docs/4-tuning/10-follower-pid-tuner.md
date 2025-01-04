---
title: "팔로워 PID 조율 (Follower PID Tuner)"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 409
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
rectangle "<b>로컬라이제이션 테스트</b>" as s8
rectangle "현재 단계" {
rectangle "<b>팔로워 PID 조율</b>" as s9
}
rectangle "<b>스플라인 테스트</b>" as s10

(s4_y) --> (s5)
(s4_n) --> (s5)
(s5) --> (s6)
(s6) --> (s7)
(s7) --> (s8)
(s8) --> (s9)
(s9) --> (s10)
```
현재 단계는 **팔로워 PID 조율** 단계입니다.

이 단계는 마지막 튜닝 단계로, 폐쇄 루프 피드백 컨트롤러를 조정하는 과정입니다. 
이를 통해 경로 추적 성능이 크게 향상될 것입니다.

이 과정을 위해 사용할 수 있는 두 가지 OpMode가 있습니다: 
1. `BackAndForth`
2. `FollowerPIDTuner`

먼저 `BackAndForth` 모드를 실행하여 대략적인 PID 게인을 설정한 뒤, 
`FollowerPIDTuner`를 사용해 세부적인 튜닝을 진행하는 것을 추천합니다.

`BackAndForth` 모드는 로봇이 직선으로 앞뒤로 이동하며, 
여기에서 병진 이동(Translation) 및 회전(Heading) PID의 주요 오류를 쉽게 확인할 수 있습니다. 

`FollowerPIDTuner`는 로봇이 큰 사각형 경로를 따라 이동하도록 설정합니다. 
이때 각 코너에서 **반시계 방향**으로 회전합니다. 
방향 조정이 부정확하면 전체 경로가 어긋나며, 
로봇을 계속 초기화해야 하므로 과정이 다소 번거로울 수 있습니다. 
따라서 초기 튜닝은 `BackAndForth` 모드에서 진행하고, 이후 세부 조정은 `FollowerPIDTuner`를 활용하는 것이 좋습니다.

---

### 조율 과정 (Tuning Process)

1. `BackAndForth` OpMode를 RC(Remote Control)에서 실행한다.

2. RC의 Wi-Fi 네트워크에 연결합니다.
   네트워크 비밀번호는 `Program and Manage` 메뉴에서 확인할 수 있습니다.

3. 브라우저에서 다음 주소로 이동합니다.
    - RC 핸드폰 사용 시: `192.168.49.1:8080/dash`
    - Control Hub 사용 시: `192.168.43.1:8080/dash`

4. 화면 우측 상단에서 `Field` 뷰를 선택했는지 확인합니다.

5. 두 개의 선과 원이 화면에 그려지는 것을 확인합니다.
    - 초록색 : 목표 위치
    - 파란색 : 로봇의 실제 위치

6. 우측 사이드바에서 `SampleMecanumDrive`를 찾고 드롭다운을 여십시오.
   `HEADING_PID`와 `TRANSLATION_PID` 두 가지 옵션이 표시됩니다. 
   두 옵션 모두 `SampleMecanumDrive` 파일에 위치합니다.

7. 먼저 `HEADING_PID`를 먼저 엽니다.
    - `kP` 값을 점진적으로 증가시키며 로봇이 정확한 헤딩을 유지하도록 조정하세요.
    - 경험적으로 `kP` 값은 약 **8** 정도였지만, 환경에 따라 달라질 수 있습니다.
    - 일반적으로 `kD`와 `kI`는 조정할 필요가 없습니다.

8. `TRANSLATION_PID`를 엽니다.
    - 마찬가지로 `kP` 값을 점진적으로 증가시키며 로봇이 경로를 잘 따라가도록 조정하세요.
    - 이 값도 약 **8** 정도였지만, 환경에 따라 달라질 수 있습니다.
    - `kD`와 `kI`는 일반적으로 조정할 필요가 없습니다.

9. 튜닝이 완료되면, 값을 `SampleMecanumDrive.java` 파일의 PID 객체에 복사하십시오. 
   대시보드에서 변경한 값이 코드에도 반영되도록 해야 합니다.

10. `FollowerPIDTuner`를 사용해 동일한 과정을 반복하며 추가적인 정밀 튜닝을 진행하세요. 
    이는 더욱 정확한 결과를 얻기 위해 권장됩니다.

11. 튜닝이 완료되었으면 `SplineTest`를 실행해 경로 추적 정확도를 확인하세요.

{{< callout context="note" title="노트" icon="outline/info-circle" >}}
- 일반적으로 `kI`와 `kD`는 필요하지 않다고 했지만, 이는 기본 가이드라인일 뿐입니다.
- `kD`는 피드포워드(Feedforward)를 사용하는 경우 배터리 전압 변화에도 포즈 속도를 일정하게 유지하는 데 도움을 줄 수 있습니다. 이 경우 `kD` 값을 **1 정도**로 설정해보세요.
- 단, 주행 속도 PID를 사용하는 경우 `kD`를 설정하면 두 PID가 서로 충돌할 수 있으므로 사용하지 마세요.
{{< /callout >}}

---

### 비공식 영상

이 영상은 임시 영상입니다. 추후 업데이트될 예정입니다.

<div class="flex justify-center">
   <iframe width="560" height="315" src="https://www.youtube.com/embed/e6k_gP2YCmc" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>
