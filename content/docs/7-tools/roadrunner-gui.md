---
title: "RoadRunner GUI"
description: "자율 주행에 도움이 되는 다른 도구들을 소개합니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 702
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---
## 개요

{{< figure
src="images/tools/rr-gui-half-compressed.jpg"
alt="Road Runner 경로 생성 GUI를 보여주는 스크린샷"
caption="Road Runner 공식 GUI"
>}}

Road Runner는 경로를 설계하기 위한 공식 GUI 플러그인을 제공합니다. 이 GUI는 경로를 yaml 파일로 내보낼 수 있으며, 해당 파일은 Road Runner 라이브러리가 읽을 수 있습니다.  
하지만, 이 GUI는 스플라인 경로만 생성할 수 있습니다. `lineTo`나 상대 이동(`strafe()`, `forward()` 등)은 지원하지 않습니다. 또한, 경로가 yaml 파일로 내보내지기 때문에 마커를 지원하지 않습니다.  
공식 GUI를 사용하기로 결정했다면, GUI에서 경로를 생성한 뒤 이를 Java 코드로 변환하여 사용하는 것을 권장합니다.

---

## 설치 방법

1. [Road Runner 최신 릴리즈 페이지](https://github.com/acmerobotics/road-runner/releases/tag/v0.5.6)로 이동합니다.
2. **Assets** 드롭다운 메뉴를 펼치고 `road-runner-gui-x.x.x.jar` 파일을 찾습니다.
3. 파일 이름을 클릭하여 `.jar` 파일을 다운로드합니다.
4. 다운로드한 `.jar` 파일을 터미널에서 실행합니다:

```bash
java -jar road-runner-gui-x.x.x.jar
```

---

## 사용 방법

1. 다운로드한 `.jar` 파일을 실행합니다.
2. "Select the location for your project"라는 작은 창이 나타납니다.
3. **Browse** 버튼을 클릭하고, FTC 코드가 있는 폴더를 선택합니다.
4. 아래와 같은 창이 나타납니다:

{{< figure
src="images/road-runner-gui/step-4-half-compressed.jpg"
alt="Road Runner GUI 메인 창"
caption="Road Runner GUI 메인 창"
>}}

5. 왼쪽 상단의 **Add** 버튼을 클릭하여 경로에 이름을 지정합니다.
6. 오른쪽 하단의 **Add** 버튼을 클릭하여 더 많은 웨이포인트를 추가합니다.
    - 웨이포인트의 좌표를 수정하세요.
7. 다양한 기능을 실험해 보세요:
    - **Interp** 드롭다운 메뉴를 통해 각 웨이포인트의 헤딩 보간(interpolation) 유형을 변경할 수 있습니다.
    - **Config** 탭에서는 로봇의 물리적 속성과 제약 조건을 변경할 수 있습니다.
8. 필드 위로 커서를 올리면, 로봇이 경로를 따라 움직이는 애니메이션을 확인할 수 있습니다.
9. **Save** 버튼을 클릭하면 경로가 YAML 파일로 내보내집니다.