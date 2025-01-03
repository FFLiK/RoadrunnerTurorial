---
title: "자주 묻는 질문 (FAQ)"
description: "로드러너에 대한 전체적인 개요를 설명하는 문서입니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 102
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

## **1. Pure Pursuit와의 차이점**
Road Runner와 Pure Pursuit는 자주 비교되지만, 본질적으로 매우 다른 도구입니다.

- **Pure Pursuit**: 비홀로노믹(Non-Holonomic) 드라이브 트레인을 위한 경로 추적 알고리즘으로, "룩어헤드 포인트(Look-Ahead Point)"를 사용해 다차원 경로를 따릅니다.
- **Road Runner**: Ramsete, 가이드 벡터 필드 등 여러 경로 추적 알고리즘을 지원하며, 속도와 가속도를 제어하는 모션 프로파일링 기능을 포함합니다.

특히, Pure Pursuit는 가속도 제어가 없기 때문에 데드 휠 오도메트리 설치가 필수입니다. 또한, FTC 로봇 대부분이 홀로노믹(메카넘) 드라이브를 사용하기 때문에 Pure Pursuit는 잘 맞지 않는 경우가 많습니다.

---

## **2. 데드 휠/오도메트리란 무엇인가요?**
데드 휠과 오도메트리는 종종 같은 의미로 사용되지만, 사실 차이가 있습니다.

- **오도메트리**: 로봇의 위치를 계산하기 위해 센서를 사용하는 기술입니다.
- **데드 휠**: 동력이 없는 옴니 휠로, 회전 인코더를 통해 이동 거리를 측정합니다. 이 데이터를 통해 로봇의 상대적 위치(x, y, 헤딩)를 계산할 수 있습니다.

메카넘 드라이브에서는 데드 휠이 슬립을 줄여 정확도를 크게 향상시킵니다.

{{< figure
src="images/home/dead-wheel-example.jpg"
alt="데드 휠의 예시"
caption="데드 휠의 예시"
>}}

---

## **3. 스플라인 경로란 무엇인가요?**
스플라인 경로는 [스플라인 곡선](<https://www.wikiwand.com/en/Spline_(mathematics)>)을 사용해 생성된 경로입니다.
- 스플라인 곡선은 여러 점을 매끄럽게 연결하는 다항식 조각으로 이루어져 있습니다.
- 연속적인 경로를 따라 이동하면서 헤딩을 변경할 수 있어, 자율 주행 경로에 이상적입니다.

Road Runner에서는 스플라인 경로를 자주 사용하며, 직선 경로를 단순히 연결하는 것보다 훨씬 부드러운 움직임을 제공합니다.

---

## **4. 기본 단위를 변경할 수 있나요?**
Road Runner는 기본적으로 인치 단위를 사용합니다.
- 공식 FAQ에 따르면, 기본 단위는 인치로 설정되어 있으며, 이를 변경하려면 Road Runner 인터페이스에 대한 래퍼를 직접 작성해야 합니다.
- 따라서 기본적으로 인치를 사용하는 것이 권장됩니다.

---

<small> 더 자세한 정보가 필요하다면 <a href="https://discord.gg/first-tech-challenge">FTC Discord</a>를 방문해 보세요! 😊 </small>
