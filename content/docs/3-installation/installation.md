---
title: "설치 및 준비"
description: "로드러너 설치 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 301
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---
Road Runner 라이브러리를 설치하는 방법에는 두 가지가 있습니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
Road Runner는 점차 레거시 소프트웨어로 전환되고 있어, 2023-24 시즌 이후에는 Quickstart가 최신 SDK 요구 사항과 호환되지 않을 수 있습니다. Driver Station에서 "로봇 컨트롤러가 오래되었습니다"라는 오류가 발생하면, [FIRST 공식 SDK](https://github.com/FIRST-Tech-Challenge/FtcRobotController)를 다운로드하고 [방법 #2](#방법-2-기존-프로젝트에-rr-설치하기)를 사용하세요.
{{</callout>}}

[방법 #1](#방법-1-quickstart-다운로드)은 더 간단한 옵션으로, [Quickstart 저장소](https://github.com/acmerobotics/road-runner-quickstart/tree/quickstart1)를 다운로드하는 것입니다. 이 저장소는 빈 FTC 시즌 프로젝트와 함께 Road Runner를 바로 실행할 수 있도록 필요한 종속성과 튜닝 관련 OpMode가 미리 설치되어 있습니다. 하지만 이미 기존 코드베이스가 있는 경우에는 이 방법이 적합하지 않을 수 있습니다.

[방법 #2](#방법-2-기존-프로젝트에-rr-설치하기)는 Gradle을 사용해 Road Runner를 설치하고 Quickstart 저장소에서 필요한 파일을 복사해 기존 팀 프로젝트에 추가하는 방법입니다.

설치 후, Rev Expansion Hub 또는 Control Hub 펌웨어를 업그레이드하는 것을 강력히 권장합니다. 자세한 방법은 [아래](#펌웨어-업그레이드)에서 확인할 수 있습니다.

---

## 방법 1: Quickstart 다운로드

1. [Quickstart 저장소](https://github.com/Iris-TheRainbow/RoadRunnerQuickstart15031)에 접속하세요. (이 저장소는 Iris_TheRainbow가 관리하며, 공식 Quickstart보다 최신 상태를 유지할 가능성이 높습니다.)
2. 녹색 "Code" 버튼을 클릭한 뒤 "Download ZIP"을 선택하세요.

<VideoDisplay src="./assets/installing/github-download-btn.mp4" width="100%"/>

3. 다운로드한 ZIP 파일을 원하는 디렉토리에 압축 해제하세요.
4. Android Studio에서 해당 폴더를 열어주세요.
5. 이제 Road Runner를 실행할 준비가 완료되었습니다!

---

## 방법 2: 기존 프로젝트에 RR 설치하기

::: warning
이 설치 가이드는 SDK 9.1 기준으로 작성되었습니다. 이후 SDK 버전에서는 정확하지 않을 수 있습니다.
:::

1. 프로젝트가 최신 FTC 표준 프로젝트 파일 구조(작성 시점 기준 SDK **9.1**)와 동일하다고 가정합니다. 해당 프로젝트는 [여기](https://github.com/FIRST-Tech-Challenge/FtcRobotController)에서 확인할 수 있습니다.

2. 프로젝트 루트 디렉토리에서 `build.dependencies.gradle` 파일을 찾으세요.

<!-- prettier-ignore -->
```plaintext
FtcRobotController
├── .github
├── FtcRobotController
├── TeamCode
├── doc
├── gradle/wrapper
├── libs
├── .gitignore
├── README.md
├── build.common.gradle
├── `build.dependencies.gradle` _(**이 파일**_)
├── build.gradle
├── gradle.properties
├── gradlew
├── gradlew.bat
└── settings.gradle
```

3. `repositories` 블록 끝에 다음 코드를 추가하세요:

   ```groovy
   maven { url = 'https://maven.brott.dev/' }
   ```

   그런 다음, `dependencies` 블록 끝에 아래 코드를 추가하세요:

   ```groovy
   implementation 'com.acmerobotics.dashboard:dashboard:0.4.15'
   ```

::: warning
이 가이드는 작성 시점(2024년 2월 17일) 기준 최신 상태입니다. 특히 2024-2025 FTC 시즌이 시작되면 [이 웹사이트](https://acmerobotics.github.io/ftc-dashboard/gettingstarted)를 방문해 최신 버전을 확인하고 업데이트된 필드 다이어그램을 사용하세요.
:::

OpenRC를 사용하는 경우, [여기](https://acmerobotics.github.io/ftc-dashboard/gettingstarted)에서 대시보드 관련 별도 지침을 확인하세요.

4. `TeamCode/build.gradle` 파일을 찾아 열어주세요.

<!-- prettier-ignore -->
```plaintext
FtcRobotController
├── .github
├── FtcRobotController
├── TeamCode
│  ├── src/main
│  └── `build.gradle` _(**이 파일**_)
├── doc
├── gradle/wrapper
├── libs
├── .gitignore
├── README.md
├── build.common.gradle
├── build.dependencies.gradle
├── build.gradle
├── gradle.properties
├── gradlew
├── gradlew.bat
└── settings.gradle
```

5. `TeamCode/build.gradle` 파일에 아래 종속성을 추가하세요:

   ```groovy
   implementation 'org.apache.commons:commons-math3:3.6.1'
   implementation 'com.fasterxml.jackson.core:jackson-databind:2.12.7'
   implementation 'com.acmerobotics.roadrunner:core:0.5.6'
   ```

6. [Quickstart 저장소](https://github.com/acmerobotics/road-runner-quickstart/tree/quickstart1)를 다운로드하세요. Git을 사용하는 경우 아래 명령어를 실행하세요:

   ```bash
   git clone --single-branch -b quickstart1 https://github.com/acmerobotics/road-runner-quickstart.git
   ```

7. `TeamCode` 폴더에 있는 `drive`, `util`, `trajectorysequence` 폴더를 프로젝트의 적절한 위치(보통 `TeamCode` 폴더)로 이동하세요.

---

## 펌웨어 업그레이드

Control Hub 또는 Expansion Hub 펌웨어를 최신 버전으로 업그레이드하는 것을 강력히 추천합니다. 펌웨어 버전 1.8.2는 다음과 같은 성능 개선 사항을 제공합니다:

- DC 모터 출력 선형성 향상
- 클로즈 루프 제어 개선
- I2C 속도 향상
- ESD 오류 복구를 위한 USB 복구 기능

Road Runner의 성능은 이러한 개선 사항의 직접적인 영향을 받습니다.

펌웨어 업그레이드 방법은 [REV 공식 문서](https://docs.revrobotics.com/rev-control-system/managing-the-control-system/updating-firmware)에서 확인할 수 있습니다.

---

**이제 준비 완료!** 설치가 끝났습니다. 이제 튜닝을 시작하세요! 🚀