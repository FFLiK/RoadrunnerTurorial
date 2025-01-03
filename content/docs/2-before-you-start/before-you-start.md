---
title: "시작하기 전에"
description: "로드러너를 직접 사용하기 전 필요한 지식을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 201
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
계속하기 전에 아래 사항을 반드시 확인하세요!
{{</callout>}}

{{<callout context="tip" title="팁" icon="outline/rocket">}}
**1.** 이 튜토리얼은 Java 프로그래밍 언어와 FTC SDK에 익숙하다는 전제하에 작성되었습니다. 둘 다 시간이 지나고 실습을 통해 경험이 쌓이면 익숙해질 수 있습니다. 지금 당장은 자신이 부족하다고 느낄지 몰라도, 코드를 작성하면서 점점 더 나아질 거예요! 👊
{{</callout>}}

{{<callout context="tip" title="팁" icon="outline/rocket">}}
**2.** 이 개념들은 어렵고 복잡하게 느껴질 수 있습니다. 이 튜토리얼은 최대한 쉽게 설명하려고 노력했지만, 모든 사람에게 맞는 것은 아닐 수 있습니다. 도움이 필요하다면, [FTC Discord](https://discord.gg/first-tech-challenge)에 많은 친절하고 똑똑한 사람들이 있으니 꼭 참여해보세요! Road Runner에 경험이 많은 사람들도 많습니다.
{{</callout>}}

{{<callout context="tip" title="팁" icon="outline/rocket">}}
**3.** 이 튜토리얼은 설치, 사용법, 팁과 트릭에 대해 다룹니다. 그러나 Road Runner의 내부 작동 방식에 대해 깊이 설명하지는 않습니다. PID 제어와 모션 프로파일링의 작동 방식을 깊이 이해하면 학습 과정이 훨씬 빨라질 것입니다. Road Runner의 [공식 퀵스타트 문서](https://acme-robotics.gitbook.io/road-runner/quickstart/introduction)를 확인해보는 것을 추천합니다. 여기에는 내부 작동 방식이 잘 설명되어 있습니다.
{{</callout>}}

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
튜닝 과정을 진행할 때 충분한 공간을 확보하는 것이 매우 중요합니다. 넉넉한 공간은 더 정확한 튜닝을 가능하게 합니다. 개인적으로 로봇이 주행할 수 있는 90인치 정도의 공간을 추천합니다. 최소한 필드 너비(72인치) 정도는 확보해야 합니다.
{{</callout>}}

<small> 이제 준비 완료! 다음 페이지로 이동해 시작해봅시다! 🚀 </small>

---

## 알아두어야 할 용어

많은 용어가 나오는데 잘 이해되지 않는다면 걱정하지 마세요. 이 페이지에서 주요 용어를 정리해 드립니다.

혹시 이 페이지에 없는 반복적으로 등장하는 용어가 있다면, Discord (Noah#5396)로 연락하거나 GitHub Pull Request를 통해 추가 요청을 해주세요!

### 로컬라이제이션 (Localization)

고급 FTC 팀들 사이에서 "로컬라이제이션"이라는 용어를 자주 듣게 될 것입니다. 로컬라이제이션이란 기본적으로 로봇이 현재 자신의 위치를 알고 있는 능력을 의미합니다. 자신의 위치를 모른 채 특정 지점으로 이동하는 것은 매우 어려운 일입니다(이는 폐쇄 루프 제어와 개방 루프 제어의 차이와 유사합니다). 로컬라이제이션은 일반적으로 오도메트리(odometry)를 통해 이루어지며, 때로는 VSLAM 같은 더 복잡한 방법(Intel Realsense T265 사용 등)을 사용할 수도 있습니다. 구동 엔코더나 외부 휠을 사용해 데이터를 수집하고, 이를 운동 방정식에 대입하여 로봇의 상대적 자세(x, y, 방향)를 계산합니다.

### 모션 프로파일 (Motion Profile)

모션 프로파일은 특정 상태에 도달하기 위해 따라야 할 동작을 그래프로 나타낸 것입니다. Road Runner의 경우, 주로 로봇의 속도를 정의된 자세로 이동시키기 위한 그래프를 생성합니다(모션 프로파일은 엘리베이터와 같은 다른 구성 요소에도 생성될 수 있습니다). 간단히 말해, 모션 프로파일은 특정 지점으로 이동하기 위해 필요한 전체 움직임을 계획하는 것입니다. 최대 속도, 최대 가속도 등을 정의하여 이 그래프를 제어할 수 있습니다. 더 자세한 설명은 [여기](https://www.motioncontroltips.com/what-is-a-motion-profile/)를 참조하세요. 또한, FRC 팀 254의 [컨퍼런스 강연](https://www.youtube.com/watch?v=8319J1BEHwM)도 훌륭한 자료입니다.

### 개방 루프 제어 vs 폐쇄 루프 제어 (Open vs Closed Loop Control)

이 둘의 차이는 피드백의 유무에 달려 있습니다. 피드백이 있으면 "루프를 닫을 수" 있습니다. 그렇다면 피드백이란 무엇일까요?

예를 들어, 모터의 속도를 제어하려고 한다고 가정해봅시다. 개방 루프 제어는 원하는 값을 "추정"하는 방식입니다. 기존의 수학적 모델을 사용해 값을 계산한 뒤, 이를 적용해 원하는 결과를 기대합니다. 하지만 현실에서는 물리적 허용 오차, 전기적 노이즈 등의 이유로 정확한 결과를 얻기 어렵습니다.

반면 폐쇄 루프 제어에서는 엔코더를 통해 실제 속도를 측정하고, 이를 기반으로 출력 전압을 조정합니다. 이 방식은 PID 컨트롤러를 통해 주로 구현됩니다.

### Vector2d

2차원 벡터를 나타냅니다: X와 Y 좌표.

```java
// (x: 10, y: -5) 좌표에 벡터 생성
Vector2d myVector = new Vector2d(10, -5);
```

### Pose2d

2차원 로봇 자세를 나타냅니다: X와 Y 좌표, 그리고 방향.

로봇의 위치와 방향을 나타내며, 각도가 증가하면 반시계 방향으로 회전합니다(삼각함수에서 배운 단위 원과 동일). 각도는 반드시 라디안 단위로 표현해야 하므로 `Math.toRadians()` 함수를 사용해 도를 라디안으로 변환합니다.

```java
// (x: 10, y: -5) 좌표에서 90도 방향을 향하는 자세 생성
Pose2d myPose = new Pose2d(10, -5, Math.toRadians(90));
```