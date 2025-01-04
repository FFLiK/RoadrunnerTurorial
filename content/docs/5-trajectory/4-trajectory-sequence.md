---
title: "경로 서열 정하기 (Trajectory Sequence)"
description: "로드러너를 사용해 자율 주행 경로를 설정하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 504
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

2021년 5월 6일 기준, 
[Road Runner quickstart](https://github.com/acmerobotics/road-runner-quickstart/tree/quickstart1)에는 
"trajectory sequences"라는 유틸리티가 포함되어 있습니다. 

경로 서열(Trajectory Sequences)는 여러 경로를 하나로 이어붙이며, 
자동으로 [경로 연속성 예외](/trajectories.html#path-continuity-exception)를 처리합니다. 
또한, 회전(Turn) 및 대기(Wait)를 "1급(First-class)" 동작으로 전환합니다(최소한 Quickstart 내에서는). 
이를 통해 여러 개의 서로 다른 Trajectory로 나누지 않고 하나의 경로 서열로 전체 자율 주행 경로를 실행할 수 있습니다.

이제 마커(marker)는 경로(Trajectory), 대기(Wait), 회전(Turn) 동작 어디에나 삽입될 수 있습니다. 
이를 통해 비동기(Async)나 동시(Concurrent) 코드를 작성하지 않고도 대기와 회전 동작 중에 작업을 실행할 수 있습니다. 
하지만 이는 간단한 작업에만 권장됩니다. 
더 복잡한 작업은 여전히 [동시 유한 상태 기계(Concurrent Finite State Machines)](/advanced.html#async-following)와 같은 방법으로 처리해야 합니다.

보너스로, 경로 서열은 더 고급 대시보드 필드 드로잉 기능도 제공합니다!

다음은 경로 서열의 개선 사항을 보여주는 단순한 예제입니다.

#### 경로 서열 사용 전
```java
public class WorldChampionshipAuto extends LinearOpMode {
    @Override
    public void runOpMode() throws InterruptedException {
        SampleMecanumDrive drive = new SampleMecanumDrive(hardwareMap);

        Pose2d startPose = new Pose2d(0, 0, 0);
        ElapsedTime timer = new ElapsedTime();

        drive.setPoseEstimate(startPose);

        Trajectory traj1 = drive.trajectoryBuilder(startPose)
                .splineTo(new Vector2d(10, 10), 0)
                .build();

        Trajectory traj2 = drive.trajectoryBuilder(startPose)
                .splineTo(new Vector2d(25, -15), 0)
                .build();

        Trajectory traj3 = drive.trajectoryBuilder(startPose)
                .forward(10)
                .build();

        // strafeRight(10)은 PathContinuityException을 발생시키므로 traj3에 포함할 수 없음
        Trajectory traj4 = drive.trajectoryBuilder(startPose)
                .strafeRight(5)
                .build();

        Trajectory traj5 = drive.trajectoryBuilder(startPose)
                .strafeRight(5)
                .build();

        Trajectory traj6 = drive.trajectoryBuilder(startPose)
                .splineToLinearHeading(new Pose2d(-10, -10, Math.toRadians(45)), 0)
                .build();

        waitForStart();

        if (isStopRequested())
          return;

        drive.followTrajectory(traj1);
        drive.turn(Math.toRadians(90));
        drive.followTrajectory(traj2);

        timer.reset();
        while(timer.seconds() < 3) drive.update();

        drive.turn(Math.toRadians(45));
        drive.followTrajectory(traj3);
        drive.followTrajectory(traj4);
        drive.turn(Math.toRadians(90));
        drive.followTrajectory(traj5);

        timer.reset();
        while(timer.seconds() < 1) drive.update();

        drive.followTrajectory(traj6);
    }
}
```

#### 경로 서열 사용 후
```java
public class WorldChampionshipAuto extends LinearOpMode {
    @Override
    public void runOpMode() throws InterruptedException {
        SampleMecanumDrive drive = new SampleMecanumDrive(hardwareMap);

        Pose2d startPose = new Pose2d(0, 0, 0);

        drive.setPoseEstimate(startPose);

        TrajectorySequence trajSeq = drive.trajectorySequenceBuilder(startPose)
                .splineTo(new Vector2d(10, 10), 0)
                .turn(Math.toRadians(90))
                .splineTo(new Vector2d(25, -15), 0)
                .waitSeconds(3)
                .turn(Math.toRadians(45))
                .forward(10)
                .strafeRight(5)
                .turn(Math.toRadians(90))
                .strafeLeft(5)
                .waitSeconds(1)
                .splineToLinearHeading(new Pose2d(-10, -10, Math.toRadians(45)), 0)
                .build();

        waitForStart();

        if (!isStopRequested())
          drive.followTrajectorySequence(trajSeq);
    }
}
```

---

## 개요

경로 서열의 내부 작동 방식은 간단합니다. 
`TrajectorySequenceBuilder`는 연속적으로 Trajectory를 생성하며 
`PathContinuityException`을 처리합니다. 
새로운 세그먼트를 생성하고 가장 최근의 세그먼트에서 시작하여 빌드합니다.

하지만 이것이 연속성 예외 문제를 해결해 주지는 않습니다. 
경로가 연속적이지 않으면 여전히 감속합니다.
이를 염두에 두고 작업해야 합니다. 
빠르게 복습이 필요하다면 [이곳](/trajectories.html#path-continuity-exception)을 확인하세요.

기본적인 Trajectory 연결은 단순히 Trajectory를 체이닝(chaining)하기 위한 
문법적 편의(Syntactic Sugar)일 뿐입니다. 
아래와 같은 간단한 큐(Queue)로 이를 구현할 수 있습니다:

```java
Queue<Trajectory> trajectoryQueue;

trajectoryQueue.add(trajectory1);
trajectoryQueue.add(trajectory2);
trajectoryQueue.add(trajectory3);
trajectoryQueue.add(trajectory4);

while(!trajectoryQueue.isEmpty()) {
    drive.followTrajectory(trajectoryQueue.poll());
}
```

이렇게 자체 경로 서열을 만들었습니다. 
하지만 이것에는 무엇이 부족할까요? 
자동 연결, 대기, 회전, 그리고 이러한 동작 안에 포함된 마커 같은 모든 멋진 기능이 빠져 있습니다. 
이를 확인해 봅시다.


<div class="flex justify-center my-8">
   <iframe width="560" height="315" src="https://www.youtube.com/embed/BF_C4szJ4vU" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

`TrajectorySequence`에 익숙해졌다면 [MeepMeep](/tool/meepmeep.html#meepmeep)을 확인하여 경로를 생성하고 시각화해 보세요!

---

## TrajectoryBuilder의 모든 기능

`TrajectorySequenceBuilder`는 `TrajectoryBuilder` API의 모든 기능을 지원합니다. 더 많은 세부 사항과 시각적 예시를 원하신다면 [TrajectoryBuilder 함수 목록](trajectorybuilder-functions.html)을 참조하세요.

---

### .turn(radians)

`TrajectorySequenceBuilder`는 일반적인 Road Runner의 회전 기능을 사용합니다. 이 구현은 `TrajectorySequenceRunner` 파일의 [140-164번째 줄](https://github.com/NoahBres/road-runner-quickstart/blob/d7d3c43715770c71bba43c45082a74803751928e/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/trajectorysequence/TrajectorySequenceRunner.java#L140)에 있습니다. 이는 이전에 `SampleMecanumDrive`에 있던 구현과 동일합니다. 이 회전 기능은 비교적 단순한 방식으로 구현되었으며, 필요하다면 타임아웃 또는 정적 피드포워드(Static Feedforward)와 같은 추가 로직으로 보완할 수 있습니다. 이러한 개선 사항은 표준화되지 않았으며, 기본 Quickstart에 병합되지 않았습니다. 더 나은 구현을 갖고 있다면 Quickstart에 Pull Request를 제안할 수 있습니다. 모든 로직이 Quickstart 측에서 처리되므로 사용자가 필요에 따라 수정할 수 있습니다.

**회전 예시 코드:**

```java
TrajectorySequence ts = drive.trajectorySequenceBuilder(startPose)
  .turn(Math.toRadians(45)) // 반시계 방향으로 45도 회전
  .build();

drive.followTrajectorySequence(ts);
```

---

### .waitSeconds(seconds)

`TrajectorySequenceBuilder`는 간단한 대기 구간을 추가하는 기능을 제공합니다. 이를 통해 트라젝토리 사이에 작업을 실행할 수 있습니다. 이 대기 구간에도 마커를 삽입할 수 있습니다.

```java
TrajectorySequence ts = drive.trajectorySequenceBuilder(startPose)
  .waitSeconds(3) // 3초 동안 대기
  .build();
```

{{< callout context="caution" title="주의" icon="outline/alert-triangle" >}}
`waitSeconds()`를 사용해야 하며, Java 객체의 기본 `wait()` 메서드를 혼동하지 않도록 주의하세요. `wait()`는 현재 스레드가 다른 스레드에 의해 `notify()` 또는 `notifyAll()`이 호출될 때까지 대기합니다. 자세한 내용은 [Oracle JavaDoc](https://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#wait())을 참조하세요. IntelliSense에서 `wait()`가 표시되더라도, 반드시 `waitSeconds()`를 사용해야 합니다.
{{< /callout >}}

---

## SequenceBuilder 고유의 마커 기능

`TrajectorySequenceBuilder`는 기존의 Trajectory에 사용되는 표준 마커들을 모두 지원합니다. 자세한 내용은 [여기](markers.html)를 참조하세요. 추가로, `TrajectorySequenceBuilder`는 몇 가지 고유 마커를 제공합니다. 이를 통해 마커가 더 강력하고 유연하게 작동합니다.

---

### .addTemporalMarker(MarkerCallback)

`addTemporalMarker(MarkerCallback)`는 `TrajectoryBuilder`에서는 존재하지 않습니다. 기본 Trajectory에서는 이 기능이 `.addDisplacementMarker(MarkerCallback)`와 동일한 역할을 하기 때문입니다. 하지만 `TrajectorySequence`에서는 이동 거리와 지속 시간이 다르기 때문에 `addTemporalMarker(MarkerCallback)`가 더 선호됩니다. 이동 거리는 회전 및 대기 구간 동안 증가하지 않지만, 지속 시간은 증가하기 때문입니다. 이 차이를 이해하지 못하면 혼란을 초래할 수 있습니다. 내부 동작 방식에 대한 시각적 설명은 [이 비디오](https://www.youtube.com/watch?v=BF_C4szJ4vU)를 참고하세요.

---

### .UNSTABLE_addTemporalMarkerOffset(offset, MarkerCallback)

이 함수는 현재 시간에 `offset`을 추가하여 마커를 설정합니다. 
`.addTemporalMarker(double, MarkerCallback)`와의 차이점은, 
해당 함수가 Trajectory 전체의 시간 기준으로 작동하는 반면, 
`.UNSTABLE_addTemporalMarkerOffset(offset, MarkerCallback)`는 호출된 지점부터의 
상대적인 시간 기준으로 작동한다는 점입니다.

```java
// 예제 1
drive.trajectorySequenceBuilder(startPose)
    .splineTo(new Vector2d(10, 10), 0)
    .addTemporalMarker(3, () -> {})
    .strafeRight(15)
    .turn(Math.toRadians(90))
    .build();

// 예제 2
drive.trajectorySequenceBuilder(startPose)
    .splineTo(new Vector2d(10, 10), 0)
    .UNSTABLE_addTemporalMarkerOffset(3, () -> {})
    .strafeRight(15)
    .turn(Math.toRadians(90))
    .build();
```

- **예제 1**: 마커는 Trajectory 전체에서 3초 후 실행됩니다.
- **예제 2**: 마커는 `splineTo()` 실행 이후 3초 후에 실행됩니다.

{{< callout context="caution" title="주의" icon="outline/alert-triangle" >}}
이 메서드는 `UNSTABLE`로 표시되며, 향후 릴리스에서 변경될 수 있습니다.
{{< /callout >}}

---

### .UNSTABLE_addDisplacementMarkerOffset(offset, MarkerCallback)

이 함수는 현재 이동 거리에 `offset`을 추가하여 마커를 설정합니다. 
`.addDisplacementMarker(double, MarkerCallback)`와의 차이점은, 
해당 함수가 Trajectory 전체의 이동 거리 기준으로 작동하는 반면, 
`.UNSTABLE_addDisplacementMarkerOffset(offset, MarkerCallback)`는 호출된 지점부터의 
상대적인 이동 거리 기준으로 작동한다는 점입니다.

```java
// 예제 1
drive.trajectorySequenceBuilder(startPose)
    .splineTo(new Vector2d(10, 10), 0)
    .addDisplacementMarker(3, () -> {})
    .strafeRight(15)
    .turn(Math.toRadians(90))
    .build();

// 예제 2
drive.trajectorySequenceBuilder(startPose)
    .splineTo(new Vector2d(10, 10), 0)
    .UNSTABLE_addDisplacementMarkerOffset(3, () -> {})
    .strafeRight(15)
    .turn(Math.toRadians(90))
    .build();
```

- **예제 1**: 마커는 Trajectory 전체에서 3인치 이동 후 실행됩니다.
- **예제 2**: 마커는 `splineTo()` 실행 이후 3인치 이동 후 실행됩니다.

{{< callout context="caution" title="주의" icon="outline/alert-triangle" >}} 
이 메서드는 `UNSTABLE`로 표시되며, 향후 릴리스에서 변경될 수 있습니다.
{{< /callout >}}

알겠습니다! 아래는 요청하신 내용을 한국어로 정확히 번역한 것입니다:

---

## 마커 사용 예시

상대적인 오프셋을 구현할 수 있는 기능은 전체 자동화 로직을 경로 서열 안에 통합할 수 있게 해주는 강력한 기능입니다. 
기존 Trajectory에서는 메커니즘 로직이 Trajectory와 완전히 분리되어 있었으며, 
Trajectory를 따르는 동안 별도로 처리해야 했습니다. 
또는 더 복잡한 동작을 위해 상태 머신(State Machine) 같은 동시 실행 스케줄링을 사용하는 것이 일반적이었습니다. 
복잡한 동작에는 여전히 상태 머신 사용이 선호되지만, **Trajectory Sequence**는 
이를 훨씬 쉽게 호출할 수 있도록 해줍니다. 기본적인 코드 예시를 살펴보겠습니다.

---

### 간단한 예제: 앞으로 이동 → 서보 내리기 → 대기 → 서보 올리기 → 옆으로 이동

다음은 이 동작을 구현하는 코드입니다:

```java
TrajectorySequence trajSeq = drive.trajectorySequenceBuilder(startPose)
    .forward(10)
    .addTemporalMarker(() -> servo.setPosition(0)) // 서보 내리기
    .waitSeconds(3)
    .addTemporalMarker(() -> servo.setPosition(1)) // 서보 올리기
    .strafeRight(5)
    .build();
```

이 코드만 보면 기존의 Trajectory로도 충분히 구현할 수 있었으며, 
조금 더 복잡하지만 크게 어렵지는 않았습니다. 
하지만 경로 서열을 사용하면 코드가 훨씬 간결하고 읽기 쉬워졌습니다.

---

### 더 복잡한 예제: 동작의 타이밍 조정

이번에는 같은 동작을 수행하지만, 
서보를 내리는 동작이 Trajectory가 끝나기 전에 시작되도록 하고, 
서보를 올리는 동작이 대기 시간이 끝나기 조금 전에 시작되도록 만들어 보겠습니다. 
이를 통해 서보가 이미 위치를 잡은 상태에서 다음 이동을 시작할 수 있습니다.

```java
TrajectorySequence trajSeq = drive.trajectorySequenceBuilder(startPose)
    .forward(10)
    .UNSTABLE_addTemporalMarkerOffset(-0.5, () -> servo.setPosition(0)) // Trajectory 끝나기 0.5초 전에 서보 내리기
    .waitSeconds(3)
    .UNSTABLE_addTemporalMarkerOffset(-0.3, () -> servo.setPosition(1)) // 대기 시간이 끝나기 0.3초 전에 서보 올리기
    .strafeRight(5)
    .build();
```

위 코드에서 첫 번째 마커는 `forward()` Trajectory의 끝에서 0.5초 전에 실행됩니다. 
두 번째 마커는 `waitSeconds()`가 끝나기 0.3초 전에 실행됩니다. 
이와 같은 타이밍 조정을 통해 동작을 더 매끄럽게 만들 수 있습니다.

---

### 서보를 여러 번 작동시키기

이번에는 앞으로 이동 → 대기 → 오른쪽으로 이동하는 동작 중, 
대기 시간 동안 서보를 두 번 작동(총 4번의 서보 동작: 내리기, 올리기, 내리기, 올리기)시키는 코드를 작성해 보겠습니다.

```java
TrajectorySequence trajSeq = drive.trajectorySequenceBuilder(startPose)
    .forward(10)
    .UNSTABLE_addTemporalMarkerOffset(0, () -> servo.setPosition(0))   // 서보 내리기
    .UNSTABLE_addTemporalMarkerOffset(0.5, () -> servo.setPosition(1)) // 서보 올리기
    .UNSTABLE_addTemporalMarkerOffset(1, () -> servo.setPosition(0))   // 서보 내리기
    .UNSTABLE_addTemporalMarkerOffset(1.5, () -> servo.setPosition(1)) // 서보 올리기
    .waitSeconds(2) // 마커는 waitSeconds() 전에 선언해야 함
    .strafeRight(5)
    .build();
```

- 첫 번째 마커는 `waitSeconds()`가 시작되자마자 실행됩니다.
- 두 번째 마커는 대기 시간 0.5초 후에 실행됩니다.
- 세 번째 마커는 대기 시간 1초 후에 실행됩니다.
- 네 번째 마커는 대기 시간 1.5초 후에 실행됩니다.

**중요:** 마커는 `waitSeconds()` **전에** 선언해야 합니다. `waitSeconds()` **후에** 마커를 선언하면, 마커는 `strafeRight()` 동안 실행됩니다. 또는 음수 오프셋을 사용해 동일한 결과를 얻을 수도 있습니다. 다음 코드는 위와 동일한 동작을 수행합니다:

```java
double waitTime = 2;

TrajectorySequence trajSeq = drive.trajectorySequenceBuilder(startPose)
    .forward(10)
    .waitSeconds(waitTime) // 이제 마커가 waitSeconds() 뒤에 배치됨
    .UNSTABLE_addTemporalMarkerOffset(-waitTime, () -> servo.setPosition(0))       // 서보 내리기
    .UNSTABLE_addTemporalMarkerOffset(-waitTime + 0.5, () -> servo.setPosition(1)) // 서보 올리기
    .UNSTABLE_addTemporalMarkerOffset(-waitTime + 1, () -> servo.setPosition(0))   // 서보 내리기
    .UNSTABLE_addTemporalMarkerOffset(-waitTime + 1.5, () -> servo.setPosition(1)) // 서보 올리기
    .strafeRight(5)
    .build();
```

이 방식에서는 마커를 `waitSeconds()` 뒤에 배치하지만, 
음수 오프셋을 사용해 대기 시간 초반으로 상대적인 타이밍을 설정합니다.

{{< callout context="caution" title="주의" icon="outline/alert-triangle" >}}
복잡한 순차 코드는 여전히 상태 머신(State Machine) 또는 기타 동시 실행 코드에서 처리하는 것이 권장됩니다.
{{< /callout >}}

---

## 기타 유용한 메서드

### `.setTangent(double)`
지정된 헤딩 접선에서 Trajectory를 시작하도록 설정합니다. 이는 `TrajectoryBuilder()` 생성자에서 커스텀 접선을 지정하는 것과 동일합니다.

### `.setReversed(boolean)`
Trajectory를 후진 상태에서 실행하도록 설정합니다. `true`로 설정하면 로봇이 뒤로 이동합니다.

### `.setConstraints(TrajectoryVelocityConstraint, TrajectoryAccelerationConstraint)`
일시적으로 속도 및 가속도 제약 조건을 재정의하여 Trajectory의 특정 구간을 빠르거나 느리게 설정할 수 있습니다.

### `.resetConstraints()`
`setConstraints()`로 설정된 임시 제약 조건을 초기화합니다. 기본 제약 조건이 다시 사용됩니다.

### `.setVelConstraint(TrajectoryVelocityConstraint)`
일시적으로 속도 제약 조건만 설정합니다.

### `.resetVelConstraint()`
임시 속도 제약 조건을 초기화합니다.

### `.setAccelConstraint(TrajectoryAccelerationConstraint)`
일시적으로 가속도 제약 조건만 설정합니다.

### `.resetAccelConstraint()`
임시 가속도 제약 조건을 초기화합니다.

### `.setTurnConstraint(maxAngVel, maxAngAccel)`
회전에 사용할 최대 각속도 및 각가속도를 설정합니다.

### `.resetTurnConstraint()`
임시 회전 제약 조건을 초기화합니다.

### `.addTrajectory(Trajectory)`
새 Trajectory를 경로 서열에 추가합니다.
