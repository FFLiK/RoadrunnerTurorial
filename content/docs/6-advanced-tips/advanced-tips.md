---
title: "추가적인 정보"
description: "로드러너를 더욱 효과적으로 사용하는 방법을 소개합니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 601
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---
고급 코드 샘플은 `advanced-examples` 브랜치에서 확인할 수 있습니다.  
[https://github.com/NoahBres/road-runner-quickstart/tree/advanced-examples](https://github.com/NoahBres/road-runner-quickstart/tree/advanced-examples)

---

## TeleOp에서 Road Runner 사용하기

TeleOp에서 Road Runner를 사용하는 데는 여러 이유가 있습니다.  
예를 들어, 복잡한 자동화를 구현하고 싶을 수도 있습니다.  
개인적으로는 **텔레오프 자동화보다는 드라이브 연습에 투자하는 것이 훨씬 더 효율적**이라고 생각합니다.  
하지만, 원한다면 자유롭게 사용해 보세요.

**TeleOp에서 Road Runner를 사용하는 주요 이유**는 로컬라이제이션(localization) 기능을 재사용해 텔레오프 중 로봇의 위치를 실시간으로 추적하는 것입니다.  
이 기능은 예를 들어 2020-21 Ultimate Goal 게임에서 링 슈터 속도를 자동 조정하는 데 활용할 수 있습니다.

> 텔레오프 자동화와 관련하여 제가 추천하지 않는 것은 **로봇이 자동으로 특정 위치로 이동해 작업을 수행하는 방식**입니다. 숙련된 드라이버가 연습을 통해 더 빠르고 정확하게 수행할 수 있습니다.  
> 하지만 **특정 물리적 메커니즘(예: 인테이킹, 리프트)을 자동화하는 것**은 매우 추천합니다.

---

### 로컬라이저(Localizer)만 사용하는 방법

로컬라이저를 사용해 로봇의 위치를 읽기만 하면 되는 경우, 불필요한 코드를 호출하지 않도록 로컬라이저만 초기화하는 것을 추천합니다.

#### 로컬라이저 사용 코드 예제
다음은 텔레오프 OpMode에서 로컬라이저를 사용하는 코드입니다:

```java
public class MyTeleopOpmode extends LinearOpMode {
    public void runOpMode() {
        // 초기화 코드 삽입

        // StandardTrackingWheelLocalizer 사용
        StandardTrackingWheelLocalizer myLocalizer = new StandardTrackingWheelLocalizer(hardwareMap);

        // 초기 위치 설정 (x: 10, y: 10, 방향: 90도)
        myLocalizer.setPoseEstimate(new Pose2d(10, 10, Math.toRadians(90)));

        waitForStart();

        while (opModeIsActive()) {
            // 반복 루프에서 항상 update() 호출
            myLocalizer.update();

            // 현재 위치 가져오기
            Pose2d myPose = myLocalizer.getPoseEstimate();

            telemetry.addData("x", myPose.getX());
            telemetry.addData("y", myPose.getY());
            telemetry.addData("heading", myPose.getHeading());

            // TeleOp 코드 삽입
        }
    }
}
```

#### 완전한 샘플 코드
전체 샘플 코드는 [여기](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/TeleOpJustLocalizer.java)에서 확인할 수 있습니다.  
이 샘플에는 아래에서 설명할 **정적 필드를 활용한 데이터 유지(persistence)** 기능도 포함되어 있습니다.

---

### SampleMecanumDrive 사용하기

`SampleMecanumDrive`의 기능을 활용하고 싶다면, 다음과 같은 방식으로 TeleOp에 추가할 수 있습니다.

#### SampleMecanumDrive 사용 코드 예제
```java
public class MyTeleopOpmode extends LinearOpMode {
    public void runOpMode() {
        // 초기화 코드 삽입

        // SampleMecanumDrive 사용
        SampleMecanumDrive drive = new SampleMecanumDrive(hardwareMap);

        // 초기 위치 설정 (x: 10, y: 10, 방향: 90도)
        drive.setPoseEstimate(new Pose2d(10, 10, Math.toRadians(90)));

        waitForStart();

        while (opModeIsActive()) {
            // 반복 루프에서 항상 update() 호출
            drive.update();

            // 현재 위치 가져오기
            Pose2d myPose = drive.getPoseEstimate();

            telemetry.addData("x", myPose.getX());
            telemetry.addData("y", myPose.getY());
            telemetry.addData("heading", myPose.getHeading());

            // 텔레오프 코드 삽입
        }
    }
}
```

#### 완전한 샘플 코드
전체 샘플 코드는 [여기](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/TeleOpDrive.java)에서 확인할 수 있습니다.  
이 샘플은 **정적 필드 데이터 유지 기능**을 포함하며, 
`SampleMecanumDrive`를 활용해 게임패드를 통한 로봇 제어를 추가한 코드입니다.

---

## OpMode 간 위치 데이터 전송

### 이유
텔레오프에서 로컬라이저를 활용하려면 **초기 위치를 정확히 설정해야 합니다**.  
초기 위치를 설정하지 않으면 프로그램은 기본적으로 `x: 0`, `y: 0`, `heading: 0`으로 간주합니다.  
따라서, **자동 모드에서 끝난 위치를 텔레오프에 전달**해야 합니다.  
이 작업은 **Java의 정적 필드(static field)**를 사용해 해결할 수 있습니다.

---

### PoseStorage 클래스 생성
먼저, 위치 데이터를 저장할 `PoseStorage` 클래스를 생성합니다.  
아래 코드처럼 **정적 필드(static field)**를 활용합니다.

```java
public class PoseStorage {
    // static 키워드를 사용하여 데이터 공유
    public static Pose2d currentPose = new Pose2d();
}
```

---

### 자동 모드에서 위치 저장
자동 모드의 마지막에 `PoseStorage.currentPose`에 현재 위치를 저장합니다.

```java
PoseStorage.currentPose = drive.getPoseEstimate();
```

---

### 텔레오프에서 위치 읽기
텔레오프 시작 시 저장된 위치를 읽어 초기 위치를 설정합니다.

```java
myLocalizer.setPoseEstimate(PoseStorage.currentPose);
```

---

### 전체 샘플 코드
다음 링크에서 전체 샘플을 확인할 수 있습니다:

- [TeleOpJustLocalizer.java](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/TeleOpJustLocalizer.java)
- [AutoTransferPose.java](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/AutoTransferPose.java)
- [PoseStorage.java](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/PoseStorage.java)

---

### 주의 사항

1. **앱 종료 시 데이터 손실**  
   앱이 갑자기 종료되면 데이터가 손실됩니다.  
   이를 방지하려면 데이터베이스 또는 파일에 위치 데이터를 저장할 수 있습니다.  
   하지만 일반적으로 이 정도로 복잡한 처리는 필요하지 않습니다.

2. **실시간 업데이트 필요**  
   현재 코드에서는 Autonomous Period가 끝날 때 한 번만 데이터를 저장합니다.  
   만약 로봇이 외부 힘으로 밀리거나 프로그램이 충돌하면 위치 정보가 정확하지 않을 수 있습니다.  
   이를 방지하려면 **업데이트 루프에서 정기적으로 데이터를 저장**하도록 구현해야 합니다.

3. **비동기 처리 활용**  
   정기적인 데이터 저장은 **비동기 처리**를 통해 구현할 수 있습니다.  
   자세한 내용은 [Finite State Machine Following](#finite-state-machine-following) 샘플을 참고하세요.

--- 

### 필드 중심 주행(Field Centric Drive)

표준 드라이브 코드 위에 필드 중심 주행을 구현하는 것은 매우 간단합니다. 
원하는 벡터를 가져와 로봇의 현재 방향(Heading)에 따라 회전시키기만 하면 됩니다. 

아래는 간단한 예제입니다:

```java
// 로봇의 현재 위치 읽기
Pose2d poseEstimate = drive.getPoseEstimate();

// 게임패드 x/y 입력으로 벡터 생성
// 그런 다음, 해당 벡터를 현재 헤딩의 역방향으로 회전
Vector2d input = new Vector2d(
        -gamepad1.left_stick_y,
        -gamepad1.left_stick_x
).rotated(-poseEstimate.getHeading());

// 회전된 입력 벡터와 오른쪽 스틱 값을 전달
// 회전 값은 회전된 입력 벡터에 포함되지 않으므로 별도로 전달해야 함
drive.setWeightedDrivePower(
        new Pose2d(
                input.getX(),
                input.getY(),
                -gamepad1.right_stick_x
        )
);
```

전체 샘플 코드는 [여기](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/TeleOpFieldCentric.java)에서 확인할 수 있습니다.

---

## 특정 지점에 맞춰 정렬(Align To Point)

이 데모는 반드시 Road Runner의 핵심 기능(모션 프로파일링 등)에 의존하지는 않지만, 
Road Runner 유틸리티를 많이 활용합니다. 
이를 자신의 코드에 Road Runner 종속 없이 포팅하는 것은 비교적 간단합니다.

이 데모는 드라이버가 "특정 지점에 맞춰 정렬" 모드로 전환하면 필드 중심 모드로 전환하고, 
지정된 지점에 로봇이 자동으로 헤딩을 조정하도록 OpMode가 헤딩 제어를 담당합니다.

OpMode 코드는 [여기](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/TeleOpAlignWithPoint.java)에서 확인할 수 있습니다.

---

## 비동기 경로 추적(Async Following)

기본적으로 Quickstart의 `followTrajectory()` 함수는 **동기적(synchronous)**으로 동작합니다. 
이는 해당 함수가 완료될 때까지 다음 줄의 코드가 실행되지 않음을 의미합니다. 
이러한 동기적 방식은 `Linear Opmode`에서 잘 동작합니다.

하지만, 리프트의 높이를 일정하게 유지하기 위해 PID 컨트롤러를 백그라운드에서 실행하고 싶다면, 
모든 코드를 단일 루프 함수에서 처리하는 반복형 OpMode를 사용해야 합니다. 
이 경우 `followTrajectory()`의 차단(Blocking) 동작은 적합하지 않습니다.

```java
public void init() {
  // 비동기 추적 메서드 호출
  drive.followTrajectoryAsync(trajectory);
}

public void loop() {
  // drive.update()를 호출하면 모든 경로 추적 로직이 처리됩니다.
  drive.update();

  // 리프트 PID 컨트롤러 업데이트 또는 기타 로직
  lift.update();
}
```

---

## 비동기 경로 연결하기(Chaining Async Trajectories)

경로를 비동기 방식으로 연결하려면 인라인 변위 마커를 사용하면 됩니다.

```java
Trajectory trajectory1;
Trajectory trajectory2;
Trajectory trajectory3;

public void init() {
  trajectory1 = drive.trajectoryBuilder(new Pose2d())
    .lineTo(new Vector2d())
    .addDisplacementMarker(() -> drive.followTrajectoryAsync(trajectory2))
    .build();

  trajectory2 = drive.trajectoryBuilder(trajectory1.end())
    .lineTo(new Vector2d())
    .addDisplacementMarker(() -> drive.followTrajectoryAsync(trajectory3))
    .build();

  trajectory3 = drive.trajectoryBuilder(trajectory2.end())
    .lineTo(new Vector2d())
    .build();

  // 초기화 시 첫 번째 경로를 설정
  drive.followTrajectoryAsync(trajectory1);
}

public void loop() {
  drive.update();
  lift.update(); // 백그라운드에서 리프트 PID 업데이트 또는 기타 작업
}
```

이 방법에는 몇 가지 단점이 있습니다:
1. 모션 프로파일 완료 후 위치를 교정하는 PID 제어가 건너뛰어집니다.
2. 이 방법으로는 지연(Delay) 또는 회전을 추가할 수 없습니다.

더 나은 구현 방법은 유한 상태 기계(Finite State Machine, FSM)를 사용하는 것입니다. 
자세한 내용은 아래를 참고하세요.

---

## 유한 상태 기계(Finite State Machine) 추적

유한 상태 기계(Finite State Machines, FSM)는 매우 간단하면서도 놀라울 정도로 강력한 패턴으로, 복잡한 동작을 쉽게 조율할 수 있게 해줍니다.
[gm0의 FSM 문서](https://gm0.copperforge.cc/en/stable/docs/software/finite-state-machines.html) 
및 [관련 영상](https://www.youtube.com/watch?v=Pu7PMN5NGkQ)을 참고하세요.

이 [샘플 코드](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/AsyncFollowingFSM.java)는 FSM을 사용해 여러 경로를 추적하며, 경로 간 대기 및 회전을 수행하는 방법을 보여줍니다.  
FSM의 비동기적인 특성 덕분에 복잡한 서브시스템을 위한 여러 FSM을 병렬로 실행하거나 사용자의 자체 로직을 백그라운드에서 실행할 수 있습니다. 
또한 매 루프마다 `PoseStorage`에 현재 위치를 저장해, 이전에 OpMode간 포즈 전송에서 언급한 데이터를 지속적으로 업데이트하는 문제를 해결합니다.
FSM을 사용하면 각 상태에서 수행할 작업과 상태 간 전환 조건을 명확하게 정의할 수 있어 로봇의 복잡한 동작을 구조적이고 확장 가능한 방식으로 관리할 수 있습니다.

---

## 허용 오차 및 타임아웃(Admissible Error and Timeout)

Road Runner의 경로는 기본적으로 시간 기반입니다. 
따라서 경로 추적 완료 여부는 내부 타이머에 의해 결정됩니다. 
그러나 추적이 완벽하지 않을 수 있기 때문에, 시간 기반 종료 조건만으로는 충분하지 않을 수 있습니다.

Road Runner의 기본 팔로워(Follower)는 
기본적으로 허용 오차(Admissible Error) 및 타임아웃(Timeout) 조건을 추가합니다. 
이는 운동 프로파일이 소진된 후 Road Runner가 추가적으로 x초를 제공하거나 
허용 오차 조건이 충족될 때까지 대기하도록 하여 경로 추적을 완료하도록 합니다. 
이 기간 동안에는 팔로워의 PID 제어기만이 로봇에 작용합니다.

기본적으로 팔로워는 0.5초의 타임아웃과 x/y 방향에서 0.5인치, 
헤딩 방향에서 5도의 허용 오차를 제공합니다. 
즉, 경로 추적이 기본적으로 위치에서 0.5인치와 헤딩에서 5도의 오차를 허용하도록 설정되어 있습니다.

정확도를 높이고 싶다면 타임아웃을 늘리고 허용 오차를 줄이는 방법을 고려할 수 있습니다. 
이러한 설정은 `SampleMecanumDrive` 클래스에서 팔로워가 초기화되는 부분에서 찾을 수 있습니다. 
두 번째에서 마지막 매개변수가 허용 오차이고, 마지막 매개변수가 타임아웃입니다.

```java
/* SampleMecanumDrive.java의 120-121행 */
follower = new HolonomicPIDVAFollower(TRANSLATIONAL_PID, TRANSLATIONAL_PID, HEADING_PID,
    new Pose2d(0.5, 0.5, Math.toRadians(5.0)), 0.5);
//   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^   ^^^
//       허용 오차 (Pose)                   타임아웃 (초)
```

허용 오차를 줄이고 타임아웃을 늘리면 정확도를 높일 수 있습니다.

#### Trajectory Following Flow
```kroki {type=PlantUML}
top to bottom direction
skinparam ranksep 30
skinparam dpi 125

card "Loop" as loop1 {
rectangle "`.followTrajectory()`" as ft
}
card "Loop" as loop2 {
rectangle "if timer.second() > timeout : finish()" as code1
rectangle "if poseError < admissibleError : finish()" as code2
}
rectangle "finish" as fin

(ft) --> (code1) : 모션 프로파일이 \n 충족될 때 까지 반복
(code1) <--> (code2) : PID 컨트롤러가 \n 조건 중 하나가 \n 충족될 때까지 작동
(code2) --> (fin)

```

---

## 180° 회전 방향 설정 (180° Turn Direction)

`drive.turn(angle)` 함수로 180° 회전을 수행할 때, 어느 방향으로 회전할지 지정할 수 있습니다.  
다른 각도는 가장 짧은 경로로 회전하지만, 180°는 방향이 모호할 수 있습니다. 
이를 해결하려면 180°에 극소 값을 더하거나 빼서 방향을 지정합니다.

```java
// 반시계 방향 회전
drive.turn(Math.toRadians(180) + 1e-6);

// 시계 방향 회전
drive.turn(Math.toRadians(180) - 1e-6);
```

---

## 경로 중단 (Interrupting a Live Trajectory)

경로를 임의로 중단해야 하는 상황이 발생할 수 있습니다. 
이러한 동작은 아래 예제에서만 적용하는 것이 적절합니다. 
이 동작을 구현하려면 **`SampleMecanumDrive`** 클래스를 수정하고 
로봇의 **`mode`**를 강제로 **`IDLE`** 상태로 변경하는 함수를 구현해야 합니다.

- [`SampleMecanumDriveCancelable`](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/SampleMecanumDriveCancelable.java)
- [`AutoBreakTrajectory`](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/AutoBreakTrajectory.java)

이 수정된 클래스들은 단순히 **`breakFollowing()`** 함수를 구현하여 주행 경로를 중단할 수 있도록 합니다. 
이 클래스들을 여러분의 프로젝트에 복사하여 사용하세요.

---

## 텔레오프에서 자동 주행 사용

**경고**: 이 샘플 코드는 Road Runner의 기능을 보여주는 데모일 뿐, 게임에서 권장되지 않습니다.

이 [샘플 코드](https://github.com/NoahBres/road-runner-quickstart/blob/advanced-examples/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/advanced/TeleOpAugmentedDriving.java)는 
TeleOp 중에 임의의 Road Runner 경로를 따라가는 방식으로 운전자 제어를 증강하는 방법을 시연합니다. 
그러나 이는 경로(Road Runner Trajectories)가 본래 의도된 목적에 부합하지 않으며, 
이와 같은 시나리오에서는 경로 추적(Path Follower)이 더 적합합니다.

이 샘플은 Road Runner의 기능을 시연하기 위한 데모로 설계되었으며, 
복잡한 행동을 생성하기 위해 "유한 상태 기계(Finite State Machine)"샘플과 
"실시간 경로 중단(Interrupting a Live Trajectory)" 샘플을 결합하는 방법을 보여줍니다.

해당 OpMode의 세부 설명은 주석에 작성되어 있습니다.

---

## 옴니 휠 (Omni Wheels)
사각형 로봇의 각 변에 옴니 휠을 하나씩 배치하면, 
이는 표준 GoBilda 방식으로 배열된 메카넘 휠과 동일한 방식으로 작동합니다. 
개념적으로, 로봇의 전면은 두 옴니 휠 사이의 중간 지점에 위치하게 됩니다. 
로봇의 모서리를 전면으로 정렬하면, 옴니 휠은 메카넘 휠과 동일한 기능을 수행하게 됩니다.