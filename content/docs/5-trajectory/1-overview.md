---
title: "개요"
description: "Road Runner를 사용해 자율 주행 경로를 설정하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 501
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

## Trajectory와 Path의 차이점

Road Runner는 경로를 구성하기 위한 **Path**와 **Trajectory** 두 가지 데이터 구조를 제공합니다.
- **Paths** : 사용자가 정의한 웨이포인트를 기반으로 생성된 전체 경로를 단순히 저장합니다. 이 경로는 직선일 수도 있고, 부드러운 스플라인(splines)일 수도 있습니다.  
- **Trajectories** : Path에 **모션 프로파일링**을 추가한 구조입니다. 즉, Trajectories는 로봇이 경로를 따라가면서 속도, 가속도, 각가속도 등을 제어할 수 있도록 합니다.

현재 Road Runner는 홀로노믹 드라이브(holonomic drive)를 위한 경로 추적(path following)을 기본 제공하지 않습니다. 
대신, 탱크 드라이브(tank drive)를 위한 Ramsete 경로 추적을 포함하고 있습니다.  
홀로노믹 드라이브를 지원하기 위해 벡터 필드 기반의 경로 추적 기능이 추가될 예정입니다. 
자세한 내용은 [FTC Discord](https://discord.gg/first-tech-challenge)를 참고하세요.  

Road Runner는 PIDVA 기반의 Trajectory Followers를 홀로노믹 드라이브와 탱크 드라이브 모두에 대해 제공합니다. 
이는 기본적으로 Quickstart에서 사용됩니다.

---

## Vector와 Pose

Vector와 Pose는 Road Runner에서 자주 사용되는 두 가지 주요 클래스입니다.

---

### **Vector2d**

2D 벡터를 나타내며, X와 Y 좌표를 포함합니다.

```java
// 좌표 (10, -5)에 벡터 생성
Vector2d myVector = new Vector2d(10, -5);
```

---

### **Pose2d**

2D 로봇 자세(Pose)를 나타내며, X, Y 좌표와 방향(Heading)을 포함합니다.
일반적으로 로봇의 위치와 방향을 나타냅니다.
방향은 삼각함수의 단위 원(Unit Circle)처럼 반시계 방향으로 증가합니다.
따라서 `turn` 함수는 반시계 방향으로 회전합니다.
각도는 **라디안** 단위로 표현되며, `Math.toRadians()`를 사용해 도(Degree)를 라디안으로 변환합니다.

```java
// 좌표 (10, -5)에서 90도 방향을 향하는 포즈 생성
Pose2d myPose = new Pose2d(10, -5, Math.toRadians(90));
```

---

## 경로 (Trajectory) 생성하기

아래 내용부터는 Road Runner Quickstart를 사용한다고 가정하도록 하겠습니다.
Quickstart를 사용하지 않더라도 기본적인 원리는 동일하며,
`SampleMecanumDrive.java`와 같은 헬퍼 함수는 무시하면 됩니다.

아래는 Trajectory를 생성하는 기본 예제입니다:

```java
public class MyOpmode extends LinearOpMode {
  @Override
  public void runOpMode() {
    SampleMecanumDrive drive = new SampleMecanumDrive(hardwareMap);

    Trajectory myTrajectory = drive.trajectoryBuilder(new Pose2d())
        .strafeRight(10)
        .forward(5)
        .build();

    waitForStart();

    if (isStopRequested()) return;

    drive.followTrajectory(myTrajectory);
  }
}
```

---

### 주요 단계 분석

#### **1. 드라이브 객체 생성**

```java
SampleMecanumDrive drive = new SampleMecanumDrive(hardwareMap);
```

`SampleMecanumDrive` 객체를 선언하고, OpMode의 `hardwareMap`을 전달합니다.

#### **2. Trajectory 빌드**

```java
Trajectory myTrajectory = drive.trajectoryBuilder(new Pose2d())
  .strafeRight(10)
  .forward(5)
  .build();
```

- **`trajectoryBuilder()` 호출:** `drive` 객체에서 호출되며, 기본 제약 조건(Constraints)을 전달합니다.
- **시작 포즈 설정:** `new Pose2d()`는 `(0, 0, 0)`의 기본값을 가집니다.  
  예를 들어, 시작 위치를 `(5, -4)`로 설정하고 90도 방향을 향하도록 하려면 `new Pose2d(5, -4, Math.toRadians(90))`로 설정합니다.
- **메서드 체이닝:** `.strafeRight(10)`은 10인치 오른쪽으로 이동, `.forward(5)`는 5인치 앞으로 이동을 의미합니다.
- **`build()` 호출:** `TrajectoryBuilder` 객체를 `Trajectory` 객체로 변환합니다.


#### **3. Trajectory 실행**

```java
waitForStart();

if (isStopRequested()) return;

drive.followTrajectory(myTrajectory);
```

`drive.followTrajectory(myTrajectory)`는 생성된 경로(Trajectory)를 로봇이 따라가도록 명령합니다.

---

## 불연속한 경로 오류 (Path Continuity Exception)

다음과 같은 Trajectory를 실행하려고 할 때:

```java
drive.trajectoryBuilder(new Pose2d())
  .strafeRight(10)
  .forward(5)
  .build();
```

프로그램이 충돌하며 `PathContinuityException` 오류가 발생할 수 있습니다. 
이는 경로가 **연속적(continuous)**이지 않기 때문입니다.

---

### 불연속성 문제

모션 프로파일링 시스템에서는 경로가 연속적이어야 합니다. 
불연속적인 경로는 물리적으로 실현 불가능하며, 순간적인 속도나 가속도의 급격한 변화가 필요하기 때문입니다.

 **문제 예시**

아래 경로에서 로봇이 핑크색 선을 따라 이동하다가 오른쪽으로 순간적으로 방향을 바꾸는 것은 불가능합니다.
로봇은 여전히 위쪽으로의 관성(momentum)을 가지고 있기 때문에, 실제로는 호(arc)를 그리며 움직이게 됩니다.

{{< figure
src="images/trajectory-overview/continuity-error-bot-quarter.jpg"
alt="불연속적인 경로"
caption="PathContinuityException 예시"
>}}

---

### **해결 방법**

#### 1. 경로를 두 개의 Trajectory로 나누기

```java
Trajectory traj1 = drive.trajectoryBuilder(new Pose2d())
  .strafeRight(10)
  .build();

Trajectory traj2 = drive.trajectoryBuilder(traj1.end())
  .forward(5)
  .build();

drive.followTrajectory(traj1);
drive.followTrajectory(traj2);
```

이 방법은 간단하지만, 각 Trajectory가 끝날 때 로봇이 멈추고 다시 시작해야 하므로 매끄럽지 않을 수 있습니다.


#### 2. 스플라인(Spline)을 사용해 연속성 유지하기

```java
Trajectory traj = drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(x1, y1), heading)
  .splineTo(new Vector2d(x2, y2), heading)
  .build();
```

스플라인은 경로를 매끄럽게 이어주는 방법으로, 로봇이 자연스럽게 움직일 수 있도록 합니다.
스플라인 경로는 시뮬레이션 도구를 사용해 결과를 확인하는 것이 좋습니다.

{{< figure
src="images/trajectory-overview/continuity-error-fix-bot-quarter.jpg"
alt="스플라인 경로"
caption="스플라인을 사용한 연속적인 경로"
>}}

---

## Pose Estimate 설정

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
자신만의 Opmode를 실행하기 전에 반드시 이 작업을 수행하세요!
{{< /callout >}}

OpMode에서 Trajectory를 실행하기 전에, 
로컬라이저(Localizer)의 위치가 
모션 프로파일(Motion Profile)의 시작 위치와 일치해야 합니다. 

기본적으로 로컬라이저의 위치는 `x: 0, y: 0`으로 설정됩니다. 
하지만 사용자 정의 시작 Pose를 모션 프로파일에 정의한 경우, 
로컬라이저의 위치가 일치하지 않아 이상한 움직임이 발생할 수 있습니다. 
로봇이 예상치 못한 방향으로 움직이거나, 위치 PID 및 헤딩 PID가 이를 보정하려 시도하면서 
비정상적인 동작을 보일 수 있습니다.

따라서 **첫 번째 모션 프로파일의 시작 위치에 맞춰 `drive.setPoseEstimate(new Pose2d())`를 OpMode에 추가**해야 합니다.

예시:

```java{hl_lines=[7]}
public void runOpMode() {
  SampleMecanumDrive drive = new SampleMecanumDrive(hardwareMap);

  // 로봇을 x: 10, y: -8, heading: 90도에서 시작하려고 합니다.
  Pose2d startPose = new Pose2d(10, -8, Math.toRadians(90));

  drive.setPoseEstimate(startPose);

  Trajectory traj1 = drive.trajectoryBuilder(startPose)
      .splineTo(new Vector2d(20, 9), Math.toRadians(45))
      .build();

  Trajectory traj2 = drive.trajectoryBuilder(traj1.end())
      .splineTo(new Vector2d(20, 9), Math.toRadians(45))
      .build();

  drive.followTrajectory(traj1);
  drive.followTrajectory(traj2);
}
```

`drive.setPoseEstimate()`는 첫 번째 Trajectory를 실행하기 전에 한 번만 호출하면 됩니다.

---

## 여러 Trajectory 실행하기

로봇이 연속적인 경로를 따르도록 설정했더라도, 
두 번째 Trajectory를 실행해야 할 경우가 있습니다. Trajectory는 연속적인 움직임을 나타내며, 
로봇이 멈추거나 방향을 전환할 때 종료됩니다.

**여러 Trajectory를 실행하는 방법:**

```java
Trajectory traj1 = drive.trajectoryBuilder(new Pose2d())
  .strafeRight(10)
  .build();

Trajectory traj2 = drive.trajectoryBuilder(traj1.end())
  .splineTo(new Vector2d(5, 6), 0)
  .splineTo(new Vector2d(9, -10), 0)
  .build();

Trajectory traj3 = drive.trajectoryBuilder(traj2.end())
  .splineTo(new Vector2d(5, 6), 0)
  .splineTo(new Vector2d(9, -10), 0)
  .build();

drive.followTrajectory(traj1);
robot.dropServo();
drive.followTrajectory(traj2);
drive.followTrajectory(traj3);
```

**중요 사항:**
1. Trajectory가 연속적으로 호출될 경우, 다음 Trajectory의 시작 Pose는 이전 Trajectory의 `end()` 값이어야 합니다.
2. `followTrajectory()`를 호출하기 전에 모든 Trajectory를 미리 생성하세요. Trajectory 생성에는 약간의 시간이 소요되므로, 실시간으로 생성하면 경로 간에 약간의 지연이 발생할 수 있습니다.
3. `robot.dropServo()`와 같은 함수는 Trajectory 실행 후 호출됩니다. 각 `drive.followTrajectory()` 함수는 동기적으로 실행되므로, 현재 Trajectory가 완료되기 전까지 다음 라인으로 진행되지 않습니다.

---

## Trajectory와 Turn의 차이점

`SampleMecanumDrive.java` 클래스는 `turn()` 함수를 지원하지만, 이 함수는 Road Runner의 핵심 기능이 아닙니다. 
단순히 회전을 위한 모션 프로파일이며, Trajectory 내에서 호출할 수 없습니다.

```java
drive.followTrajectory(traj1);
drive.turn(Math.toRadians(90));
drive.followTrajectory(traj2);
drive.turn(Math.toRadians(-270));
```

---

## 후진하기

로봇을 후진시키고 싶다면, `TrajectoryBuilder`에서 간단히 설정할 수 있습니다.

```java{hl_lines=[1]}
Trajectory trajectory = drive.trajectoryBuilder(new Pose2d(), true)
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))
  .build();
```

위의 `true`는 로봇이 경로를 후진으로 따르도록 설정합니다.

---

## Turn 후 Trajectory 실행

`drive.turn()` 호출 후 Trajectory를 실행할 경우, 
이전 Trajectory의 `end()` 값이 현재 Pose와 일치하지 않을 수 있습니다. 
이를 해결하려면, 적절한 회전을 Trajectory 시작 Pose에 추가해야 합니다.

```java{hl_lines=[6]}
Trajectory traj1 = drive.trajectoryBuilder(startPose, false)
  .forward(10)
  .build();

// 90도 회전을 traj1.end() Pose에 추가
Trajectory traj2 = drive.trajectoryBuilder(traj1.end().plus(new Pose2d(0, 0, Math.toRadians(90))), false)
  .strafeLeft(10);
  .build();

drive.followTrajectory(traj1);
drive.turn(Math.toRadians(90));
drive.followTrajectory(traj2);
```

---

## Trajectory 속도 줄이기

Trajectory의 특정 구간에서 속도를 줄이고 싶다면 다음과 같이 설정합니다:

```java{hl_lines=[3,4,11,12,21,22]}
.splineTo(
  new Vector2d(30, 30), Math.toRadians(90),
  SampleMecanumDrive.getVelocityConstraint(slowerVelocity, DriveConstants.MAX_ANG_VEL, DriveConstants.TRACK_WIDTH),
  SampleMecanumDrive.getAccelerationConstraint(DriveConstants.MAX_ACCEL)
)
```

---

## 좌표 시스템

FTC 좌표 시스템은 필드의 중심을 `(0, 0)`으로 설정합니다.
- **X축:** Red Alliance Station과 평행하며, 오른쪽으로 증가합니다.
- **Y축:** Red Alliance Station과 수직이며, 필드 안쪽으로 증가합니다.

아래 이미지는 2019/20 Skystone 필드의 좌표 시스템을 보여줍니다.

{{< figure
src="images/trajectory-overview/field-w-axes-half.jpg"
alt="2019/20 Skystone 필드 좌표 시스템"
caption="2019/20 Skystone 필드 좌표 시스템"
>}}

이 좌표 시스템의 명세는 [여기](https://github.com/ftctechnh/ftc_app/files/989938/FTC_FieldCoordinateSystemDefinition.pdf)에서 확인할 수 있습니다.  