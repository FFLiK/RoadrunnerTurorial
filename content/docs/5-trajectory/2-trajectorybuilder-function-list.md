---
title: "TrajectoryBuilder 함수 목록"
description: "로드러너를 사용해 자율 주행 경로를 설정하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 502
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

**TrajectoryBuilder** 함수 목록은 Road Runner에서 로봇 경로를 정의하고 
커스터마이징하기 위한 다양한 메소드를 제공합니다. 
아래는 주요 함수와 사용 사례의 요약입니다:

---

### **기본 직선 이동**
1. **forward(distance: Double)**
    - 로봇을 지정된 거리(인치)만큼 앞으로 이동시킵니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .forward(40)
        .build();
      ```

    ![](videos/trajectory-builder-function/forward.gif)

2. **back(distance: Double)**
    - 로봇을 지정된 거리(인치)만큼 뒤로 이동시킵니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .back(40)
        .build();
      ```

    ![](videos/trajectory-builder-function/back.gif)

---

### **측면 이동**
3. **strafeLeft(distance: Double)**
    - 로봇을 지정된 거리(인치)만큼 왼쪽으로 이동시킵니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .strafeLeft(40)
        .build();
      ```

   ![](videos/trajectory-builder-function/strafe-left.gif)

4. **strafeRight(distance: Double)**
    - 로봇을 지정된 거리(인치)만큼 오른쪽으로 이동시킵니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .strafeRight(40)
        .build();
      ```

    ![](videos/trajectory-builder-function/strafe-right.gif)

---

### **지정된 좌표로 이동**
5. **strafeTo(endPosition: Vector2d)**
    - 로봇이 지정된 좌표로 이동합니다. 이동 중 시작 시의 헤딩(방향)을 유지합니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .strafeTo(new Vector2d(40, 40))
        .build();
      ```

    ![](videos/trajectory-builder-function/line-to.gif)

6. **lineTo(endPosition: Vector2d)**
    - `strafeTo`와 동일하게 작동하며, 지정된 좌표로 이동하면서 시작 헤딩을 유지합니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .lineTo(new Vector2d(40, 40))
        .build();
      ```

   ![](videos/trajectory-builder-function/line-to.gif)

7. **lineToConstantHeading(endPosition: Vector2d)**
    - `strafeTo` 및 `lineTo`와 기능적으로 동일합니다. 지정된 좌표로 이동하며 시작 헤딩을 유지합니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .lineToConstantHeading(new Vector2d(40, 40))
        .build();
      ```

   ![](videos/trajectory-builder-function/line-to.gif)

---

### **방향(Heading)을 변경하며 이동**
8. **lineToLinearHeading(endPose: Pose2d)**
    - 로봇이 지정된 좌표로 이동하며 시작 헤딩에서 끝 헤딩으로 선형적으로 변환합니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .lineToLinearHeading(new Pose2d(40, 40, Math.toRadians(90)))
        .build();
      ```

   ![](videos/trajectory-builder-function/line-to-linear-heading.gif)

9. **lineToSplineHeading(endPose: Pose2d)**
    - 로봇이 지정된 좌표로 이동하며 시작 헤딩에서 끝 헤딩으로 곡선적으로 변환합니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .lineToSplineHeading(new Pose2d(40, 40, Math.toRadians(90)))
        .build();
      ```

   ![](videos/trajectory-builder-function/line-to-spline-heading.gif)

---

### **스플라인 경로를 따라 이동**
10. **splineTo(endPosition: Vector2d, endTangent: Double)**
    - 로봇이 스플라인 경로를 따라 지정된 좌표로 이동하며 끝 방향을 지정합니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .splineTo(new Vector2d(40, 40), Math.toRadians(0))
        .build();
      ```

    ![](videos/trajectory-builder-function/spline-to.gif)

11. **splineToConstantHeading(endPosition: Vector2d, endTangent: Double)**
    - 로봇이 스플라인 경로를 따라 이동하며 시작 헤딩을 유지합니다. 끝 방향은 스플라인 경로의 형태에 영향을 줍니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .splineToConstantHeading(new Vector2d(40, 40), Math.toRadians(0))
        .build();
      ```

    ![](videos/trajectory-builder-function/spline-to-constant-heading.gif)

12. **splineToLinearHeading(endPose: Pose2d, endTangent: Double)**
    - 로봇이 스플라인 경로를 따라 이동하며 헤딩은 선형적으로, 경로는 스플라인 형태로 변환됩니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .splineToLinearHeading(new Pose2d(40, 40, Math.toRadians(90)), Math.toRadians(0))
        .build();
      ```

    ![](videos/trajectory-builder-function/spline-to-linear-heading.gif)

13. **splineToSplineHeading(endPose: Pose2d, endTangent: Double)**
    - 로봇이 스플라인 경로를 따라 이동하며 헤딩과 경로 모두 곡선적으로 변환됩니다.
    - 예시:
      ```java
      new TrajectoryBuilder(new Pose2d())
        .splineToSplineHeading(new Pose2d(40, 40, Math.toRadians(90)), Math.toRadians(0))
        .build();
      ```

    ![](videos/trajectory-builder-function/spline-to-spline-heading.gif)

--- 

위 함수들은 로봇의 경로를 정밀하게 제어할 수 있도록 설계되었습니다. 
필요에 따라 직선, 곡선, 방향 전환 등을 조합하여 다양한 경로를 생성할 수 있습니다.