---
title: "RRPathGen"
description: "자율 주행에 도움이 되는 다른 도구들을 소개합니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 704
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

![21511이 개발한 RRPathGen](images/tools/RRPathGen.gif)

RRPathGen은 팀이 GUI를 통해 경로를 빠르게 생성할 수 있도록 돕는 프로그램입니다. 주요 기능은 다음과 같습니다:

- 경로를 가져와 시각화
- GUI 또는 텍스트 필드에서 값을 변경하여 경로 수정
- Java 코드 형식으로 경로 내보내기
- 로봇의 길이와 너비 사용자 지정
- 다양한 화면 크기 및 해상도 지원
- 필드의 다른 위치에서 로봇이 시작하도록 경로를 뒤집는 기능
- MeepMeep과 동일한 필드 좌표계 사용
- 곧 `TrajectorySequence` 지원 예정

---

## 설치 방법 (Jar 파일)

1. [릴리즈 페이지](https://github.com/Jarhead20/RRPathGen/releases)에서 `.jar` 파일 다운로드
2. 최소 Java 8 이상이 설치되었는지 확인
   ```bash
   java --version
   ```
3. `.jar` 파일을 실행
    - 더블 클릭으로 실행하거나, 명령줄에서 실행
      
   ```bash
    java -jar RRPathGen-X.X.X.jar
    ```

---

## 설치 방법` (IntelliJ)`

1. 저장소 클론
   ```bash
   git clone https://github.com/Jarhead20/RRPathGen.git
   ```
2. 실행 구성 설정 : IntelliJ에서 `Run Configuration` 생성
3. 앱 실행

---

## 사용 방법

1. 아래 키 바인딩을 사용해 경로 생성
2. 생성이 완료되면 **내보내기(export)** 버튼 클릭
3. 내보낸 경로를 복사하여 자율 주행 프로그램에 붙여넣기

| 키 바인드             | 동작                     |
|-----------------------|--------------------------|
| **Left Click**        | 새 포인트 추가           |
| **Left Drag (Point)** | 선택한 포인트 드래그     |
| **Alt + Left Click**  | 헤딩(heading) 변경       |
| **Left Arrow**        | 다음 경로로 이동         |
| **Right Arrow**       | 이전 경로로 이동         |
| **R**                 | 로봇 방향 반전           |
| **Delete**            | 선택한 노드 삭제         |
| **Ctrl + Z**          | 이전 작업 실행 취소      |

### ⚠️ 설정 복구
- 설정 파일을 잘못 변경한 경우, 아래 경로에서 파일 삭제:
    - **Windows**: `%appdata%/RRPathGen`
    - **MacOS**: `~/Library/Application Support/RRPathGen/config.properties`
    - **Linux**: `~/.RRPathGen/config.properties`

---

## 감사의 말

- **프로젝트 영감**: Technic Bots의 [Blitz 앱](https://technicbots.com/Blitz)에서 영감을 받음
- **필드 이미지**: [MeepMeep](https://github.com/NoahBres/MeepMeep)에서 제공
- **스플라인 구현 지원**: [Ryan Brott](https://github.com/rbrott)