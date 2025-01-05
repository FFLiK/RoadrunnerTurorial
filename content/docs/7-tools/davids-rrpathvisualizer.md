---
title: "David's RRPathVisualizer"
description: "자율 주행에 도움이 되는 다른 도구들을 소개합니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 703
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

{{< figure
src="images/tools/rrpathviz-half-compressed.jpg"
alt="David의 RRPathVisualizer를 보여주는 스크린샷"
caption="David의 RRPathVisualizer"
>}}

RRPathVisualizer는 Recharged Green 7236 팀의 리드 프로그래머인 David가 작성한 경로 "시각화 도구"입니다. 
이 도구는 로봇용 경로를 작성하듯이 경로를 작성한 뒤, 이를 커스텀 Kotlin 프로젝트에 추가하는 방식으로 작동합니다. 
프로그램을 실행하면 창이 열리며, 지정된 경로를 따라 로봇이 움직이는 애니메이션을 표시합니다.  
개인적으로는 이 워크플로우를 선호하는데, RRPathVisualizer에서 작성한 경로를 간단히 복사하여 
자신의 FTC 프로젝트에 붙여넣을 수 있기 때문입니다.

참고로, RRPathVisualizer는 Kotlin으로 작성되었습니다. 
Java를 알고 있다면 Kotlin을 이해하는 데 큰 어려움이 없을 것입니다.

{{< callout context="note" title="노트" icon="outline/info-circle" >}}
David의 RRPathVisualizer와 Road Runner 공식 GUI는 모두 회전된 필드 좌표계를 사용합니다. 
두 프로그램의 필드는 관중의 관점을 기준으로 90도 회전되어 있으며, 이로 인해 Y축은 수평이고 X축은 수직입니다. 
Y축 값은 왼쪽으로 증가하고, X축 값은 위로 증가합니다. 
두 애플리케이션을 사용할 때 혼란을 방지하기 위해 이 점을 유의하세요.

MeepMeep은 회전된 필드를 사용하지 않습니다. 
관중의 관점에서 필드가 표시되지는 않지만, X축과 Y축은 전형적인 데카르트 좌표계를 따릅니다. 
더 자세한 내용은 [좌표계 명세서](/trajectories.html#coordinate-system)를 참조하세요.
{{< /callout >}}

---

### 설치 방법

1. [Intellij](https://www.jetbrains.com/idea/) 설치
    - 커뮤니티 에디션은 무료입니다.
    - [학생 계정](https://www.jetbrains.com/community/education/#students)으로 Ultimate Edition을 무료로 사용할 수도 있습니다.

2. [RRPathVisualizer](https://github.com/RechargedGreen/RRPathVisualizer)를 클론하거나 다운로드
3. Intellij에서 프로젝트 열기

---

### 사용 방법

1. Intellij에서 프로젝트 열기
    - Android Studio와 유사한 인터페이스이므로 익숙할 것입니다.

2. 상단의 **재생 버튼**을 눌러 실행
    - 정상적으로 실행되어야 합니다.

{{< figure
src="images/rrpathviz/step-2-half-compress.jpg"
alt="Intellij에서 열린 RRPathVisualizer 스크린샷"
caption="Intellij에서 열린 RRPathVisualizer 스크린샷"
>}}

3. 프로젝트 SDK 설정이 필요할 수 있음
    - **File > Project Structure**로 이동
    - "Project SDK" 설정에서 최신 JDK 버전을 선택
    - Intellij는 JDK 14를 기본적으로 포함

4. `TrajectoryGen.kt` 파일 열기
5. `driveConstraints` 값이 로봇의 Road Runner 설정에서 사용하는 `DriveConstraints`와 일치하는지 확인
6. `trackWidth` 값이 `DriveConstants.java` 파일의 `TRACK_WIDTH`와 일치하는지 확인
    - 이는 경로 길이 추정치를 보다 정확하게 만듭니다.

7. `builder1` 찾기
    - `builder1` 객체는 제공된 예제 `TrajectoryBuilder`입니다.
    - 추가 경로를 생성하려면 더 많은 builder를 자유롭게 추가 가능

8. 일반적으로 경로를 생성하듯이 builder 사용
9. `list.add(trajectory)`를 호출하여 더 많은 경로 추가 가능
10. **참고:** RRPathVisualizer는 **포인트 턴(point turns)**을 시뮬레이션할 수 없습니다.

---

### 주의 사항

실제 로봇에서는 각 경로 완료 후 0.5초의 타임아웃이 설정되어 있습니다.
- 이는 경로를 이탈한 경우 **PID**를 통해 이동 및 헤딩 보정을 허용하기 위함입니다.
- 로봇이 경로를 벗어나지 않고 도달하면 이 타임아웃은 더 빨리 종료됩니다.
- 따라서 각 경로에 최대 0.5초가 추가될 수 있으며, 이 추가 시간은 RRPathVisualizer의 경로 시간 추정치에 반영되지 않습니다.
- 타임아웃 지속 시간을 변경하려면, `SampleMecanumDrive.java`의 `HolonomicPIDVAFollower`에 설정된 마지막 매개변수(기본값 0.5)를 수정하세요.

---

### 참고 자료

팀 **Bots in Black**(16633)의 RRPathVisualizer 설치, 기본 문제 해결 및 사용법 설명 영상:

<div class="flex justify-center">
   <iframe width="560" height="315" src="https://www.youtube.com/embed/70mOwbp6ANs" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>