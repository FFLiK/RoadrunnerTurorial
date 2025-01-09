---
title: "데드 휠 설정 (Dead Wheels)"
description: "Road Runner를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 403
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
데드 휠을 사용하지 않는 경우, 이 섹션은 건너뛰세요.
{{< /callout >}}

```kroki {type=PlantUML}

skinparam ranksep 30
skinparam dpi 125
 
rectangle "<b>드라이브 상수 설정</b>" as s1
rectangle "현재 단계" {
rectangle "<b>로컬라이제이션 테스트</b>" as s3
rectangle "<b>데드 휠 설정</b>" as s2
}
rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}

(s1) --> (s2)
(s2) --> (s3)
(s3) --> (s4_y)
(s3) --> (s4_n)
```
현재 단계는 **드라이브 상수 설정** 단계입니다.

구성은 데드 휠이 2개인지 3개인지에 따라 달라집니다. 차이를 모르겠다면 [FAQ](/#what-is-the-difference-between-two-and-three-wheel-odometry)를 참고하세요.

- 두 개의 데드 휠을 사용하는 경우, [Two-Wheel Odometry](#오도메트리를-2개-사용하는-경우-two-wheel-odometry) 섹션만 읽으세요.
- 세 개의 데드 휠을 사용하는 경우, [Three-Wheel Odometry](#오도메트리를-3개-사용하는-경우-three-wheel-odometry) 섹션만 읽으세요.

---

## 오도메트리를 2개 사용하는 경우 (Two-Wheel Odometry)

두 휠 구성을 사용하는 경우, 방향(heading) 소스로 자이로스코프를 사용합니다. 기본적으로 REV Hub의 내장 BNO055 자이로스코프가 사용됩니다.

다른 자이로스코프를 사용하려면, `SampleMecanumDrive.java` 파일에서 선언하고 `getRawExternalHeading()` 함수를 오버라이드하세요.

[이 파일](https://github.com/NoahBres/road-runner-quickstart/blob/master/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/TwoWheelTrackingLocalizer.java)을 다운로드한 후, `TeamCode` 폴더에 저장하세요. 파일 정리를 위해 `StandardTrackingWheelLocalizer.java` 파일 근처에 저장하는 것을 추천합니다.

다운로드한 `TwoWheelTrackingLocalizer.java` 파일을 열어 수정하세요.

---

### 회전당 틱 수 (TPR) / 바퀴 반지름 / 기어비 설정

```java
/* TwoWheelTrackingLocalizer.java 파일의 37-39번째 줄 */
public static double TICKS_PER_REV = 0;
public static double WHEEL_RADIUS = 2; // 인치 단위
public static double GEAR_RATIO = 1; // 출력(바퀴 속도) / 입력(엔코더 속도)
```

- **`TICKS_PER_REV`**: 엔코더가 한 바퀴 회전할 때 계산하는 "틱" 수입니다. 엔코더 사양은 제조사 웹사이트에서 확인하세요. `Counts Per Revolution` 또는 `CPR` 항목을 참고하세요. REV Through Bore Encoder의 경우 `TICKS_PER_REV` 값은 `8192`입니다.
- **`WHEEL_RADIUS`**: 데드 바퀴의 반지름입니다. 지름이 아닌 반지름 값을 입력하세요.
- **`GEAR_RATIO`**: 출력(바퀴 속도)과 입력(엔코더 속도)의 비율입니다. 엔코더에 기어비를 설정하지 않았다면 `1`로 설정하세요.

---

### 수직/수평 X/Y 설정

```java
/* TwoWheelTrackingLocalizer.java 파일의 41-45번째 줄 */
public static double PARALLEL_X = 0; // 수평 X, X는 전후 방향
public static double PARALLEL_Y = 0; // 수평 Y, Y는 좌우 방향

public static double PERPENDICULAR_X = 0; // 수직 X, X는 전후 방향
public static double PERPENDICULAR_Y = 0; // 수직 Y, Y는 좌우 방향
```

수직 및 수평 바퀴의 X/Y 좌표를 입력하세요. X축은 전후 방향, Y축은 좌우 방향입니다.
이는 로봇 공학 및 항공 분야에서 일반적인 표준입니다.

{{< figure
src="images/dead-wheels/andrew-bot-wheel-location-quarter.jpg"
alt="로봇 하단 뷰. Y 방향은 좌측에서 우측으로 증가. X 방향은 상단으로 증가."
caption="17508 Rising Tau의 2019/20 Skystone 로봇"
>}}

---

### 엔코더 방향 설정

경우에 따라 `Encoder.setDirection(Encoder.Direction.REVERSE)`를 사용해 엔코더의 방향을 반전시켜야 합니다.

예시:

```java{hl_lines=[7-9]}
/* TwoWheelTrackingLocalizer.java 파일의 60-63줄 */
parallelEncoder = new Encoder(hardwareMap.get(DcMotorEx.class, "parallelEncoder"));
perpendicularEncoder = new Encoder(hardwareMap.get(DcMotorEx.class, "perpendicularEncoder"));

// TODO: Encoder.setDirection(Encoder.Direction.REVERSE)로 엔코더를 반대로 설정하세요.
// 예시) 수직 엔코더의 방향을 반대로 설정해야 하는 경우:
perpendicularEncoder.setDirection(Encoder.Direction.REVERSE);
```

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
Rev Through Bore 엔코더를 사용하는 경우 아래 내용을 반드시 참조하세요.
{{< /callout >}}

**Rev Through Bore 엔코더 주의사항**

REV Through Bore 엔코더를 사용하는 경우, 속도가 32767 cps (counts per second)를 초과하면
정수 오버플로우가 발생할 수 있습니다. 이는 REV Hub 펌웨어가 속도 데이터를 16비트 정수로 전송하기 때문입니다.
이를 해결하려면 `getRawVelocity()` 대신 `getCorrectedVelocity()`를 사용하세요.

```java{hl_lines=[5,6]}
/* TwoWheelTrackingLocalizer.java 파일의 86-95줄 */
public List<Double> getWheelVelocities() {
    //TODO : 인코더 속도가 초당 32767 카운트를 초과할 경우, Encoder.getRawVelocity()를 Encoder.getCorrectedVelocity()로 변경하여 보정 메서드를 활성화하세요.
    return Arrays.asList(
            encoderTicksToInches(parallelEncoder.getCorrectedVelocity()),
            encoderTicksToInches(perpendicularEncoder.getCorrectedVelocity())
    );
}
```

---

### 하드웨어 ID 확인

```java
/* TwoWheelTrackingLocalizer.java 파일의 60-61줄 */
parallelEncoder = new Encoder(hardwareMap.get(DcMotorEx.class, "parallelEncoder"));
perpendicularEncoder = new Encoder(hardwareMap.get(DcMotorEx.class, "perpendicularEncoder"));
```

여기서 사용한 ID가 REV Hub 설정의 ID와 일치하는지 확인하세요.

---

### IMU 설정

```java
/* SampleMecanumDrive.java 파일의 134-137줄 */
imu = hardwareMap.get(BNO055IMU.class, "imu");
BNO055IMU.Parameters parameters = new BNO055IMU.Parameters();
parameters.angleUnit = BNO055IMU.AngleUnit.RADIANS;
imu.initialize(parameters);
```

IMU가 `SampleMecanumDrive.java` 클래스에서 초기화되어 있는지 확인하세요. Quickstart를 다운로드했으며 REV Hub IMU를 사용 중이라면 변경할 필요가 없습니다. 외부 자이로를 사용하는 경우, 이 섹션을 수정하세요.

---

### IMU 속도 축 설정

`SampleMecanumDrive.java` 파일 하단의 `getExternalHeadingVelocity` 함수에서 함수가 IMU가 회전하는 축을 구성에 맞게 반환하는지 확인하세요.
파일에 제공된 ASCII 다이어그램을 참조하여 어떤 축을 선택해야 하는지 확인할 수 있습니다.
REV Hub가 평평하게 장착되어 있으면 로봇은 Z축을 중심으로 회전합니다.
모터 포트가 위 또는 아래를 향하고 있는 쪽에 있으면 로봇은 Y축을 중심으로 회전합니다.
서보 포트가 위 또는 아래를 향하고 있으면 로봇이 X축을 중심으로 회전합니다.

```java{hl_lines=[22]}
/* SampleMecanumDrive.java 파일의 399-419줄 */
@Override
public Double getExternalHeadingVelocity() {
    // TODO: 설정에 맞게 이 값을 변경해야 합니다
    //                           | Z 축
    //                           |
    //     (모터 포트 측면)       |   / X 축
    //                       ____|__/____
    //          Y 축       / *   | /    /|   (IO 측면)
    //          _________ /______|/    //      I2C
    //                   /___________ //     디지털
    //                  |____________|/      아날로그
    //
    //                 (서보 포트 측면)
    //
    // 양의 X 축은 USB 포트가 있는 방향을 가리킵니다.
    //
    // 축 회전 속도를 필요에 따라 조정하세요.
    // REV Hub/Control Hub가 평평한 표면에 놓여 있다고 가정하면
    // 기본 설정은 Z 축을 기준으로 회전합니다.
    
    return (double) imu.getAngularVelocity().zRotationRate;
}
```

### SampleMecanumDrive에서 로컬라이저(Localizer) 설정하기

로컬라이저 구성을 완료한 후, 다시 `SampleMecanumDrive.java` 파일로 돌아갑니다.

약 168번째 줄을 확인하세요. "`// TODO: if desired, use setLocalizer() to change the localization method`"라는 주석을 찾을 수 있을 것입니다.

이 주석 아래에 다음 줄을 추가하세요:

```java{hl_lines=[6]}
/* 약 168번째 줄 - SampleMecanumDrive.java */

// TODO: if desired, use setLocalizer() to change the localization method
// for instance, setLocalizer(new ThreeTrackingWheelLocalizer(...));

setLocalizer(new TwoWheelTrackingLocalizer(hardwareMap, this));
```

이렇게 로컬라이저을 완료할 수 있습니다.

---

## 오도메트리를 3개 사용하는 경우 (Three-Wheel Odometry)

세 바퀴 오도메트리 설정을 선택한 경우, 헤딩(heading)의 소스로 두 개의 평행 바퀴를 사용하게 됩니다.

`StandardTrackingWheelLocalizer.java` 파일을 엽니다.

### 회전당 틱 수 (TPR) / 바퀴 반지름 / 기어비 설정

```java
/* StandardTrackingWheelLocalizer.java - 30~32번째 줄 */
public static double TICKS_PER_REV = 0;
public static double WHEEL_RADIUS = 2; // 인치 단위
public static double GEAR_RATIO = 1; // 출력(바퀴) 속도 / 입력(엔코더) 속도
```

- **`TICKS_PER_REV`**: 인코더가 한 바퀴 회전 시 카운트하는 "틱(tick)"의 개수입니다. 인코더 사양은 제조사의 사이트에서 확인할 수 있습니다. `Counts Per Revolution`(CPR) 값을 찾으세요. 예를 들어, REV Through Bore Encoder의 경우 `TICKS_PER_REV`는 `8192`입니다.
- **`WHEEL_RADIUS`**: 데드 휠의 반지름입니다. 반드시 반지름(radius)을 사용하고 직경(diameter)을 사용하지 않도록 주의하세요.
- **`GEAR_RATIO`**: 출력(바퀴) 속도와 입력(엔코더) 속도의 비율입니다. 엔코더에 기어를 사용하지 않는 경우 `1`로 유지하세요.

---

### 측면 거리 (Lateral Distance)와 전방 오프셋 (Forward Offset) 설정

```java
/* StandardTrackingWheelLocalizer.java - 34~35번째 줄 */
public static double LATERAL_DISTANCE = 10; // 인치 단위, 좌우 바퀴 간 거리
public static double FORWARD_OFFSET = 4; // 인치 단위, 중심 축에서 전면 바퀴까지의 오프셋 거리
```

- **`LATERAL_DISTANCE`**: 좌우 바퀴 간의 거리입니다.
- **`FORWARD_OFFSET`**: 회전 중심에서 중앙 휠까지의 거리입니다.
    - 중앙 휠이 바퀴보다 앞에 있다면 양수, 뒤에 있다면 음수 값을 사용합니다.

{{< figure
src="images/dead-wheels/andrew-bot-forward-offset-quarter.jpg"
alt="로봇 하단 뷰. Y축은 왼쪽에서 오른쪽으로 증가하며, X축은 위쪽으로 증가합니다."
caption="17508 Rising Tau의 2019/20 Skystone 로봇"
>}}

---

### 엔코더 방향 설정

경우에 따라 `Encoder.setDirection(Encoder.Direction.REVERSE)`를 사용해 엔코더의 방향을 반전시켜야 합니다.

예시:
```java{hl_lines=[8-10]}
/* StandardTrackingWheelLocalizer.java - 46~63번째 줄 */
leftEncoder = new Encoder(hardwareMap.get(DcMotorEx.class, "leftEncoder"));
rightEncoder = new Encoder(hardwareMap.get(DcMotorEx.class, "rightEncoder"));
frontEncoder = new Encoder(hardwareMap.get(DcMotorEx.class, "frontEncoder"));

// TODO: 인코더 방향을 바꿀 때는 "Encoder.setDirection(Encoder.Direction.REVERSE)"를 사용.
// 예시) 중앙 인코더를 반전시켜야 할 경우:
frontEncoder.setDirection(Encoder.Direction.REVERSE);
```
이 코드를 통해 엔코더의 방향을 올바르게 설정할 수 있습니다.


{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
Rev Through Bore 엔코더를 사용하는 경우 아래 내용을 반드시 참조하세요.
{{< /callout >}}

**Rev Through Bore 엔코더 주의사항**

REV Through Bore 엔코더를 사용하는 경우, 속도가 32767 cps (counts per second)를 초과하면
정수 오버플로우가 발생할 수 있습니다. 이는 REV Hub 펌웨어가 속도 데이터를 16비트 정수로 전송하기 때문입니다.
이를 해결하려면 `getRawVelocity()` 대신 `getCorrectedVelocity()`를 사용하세요.

```java{hl_lines=[5,6]}
/* TwoWheelTrackingLocalizer.java 파일의 86-95줄 */
public List<Double> getWheelVelocities() {
    //TODO : 인코더 속도가 초당 32767 카운트를 초과할 경우, Encoder.getRawVelocity()를 Encoder.getCorrectedVelocity()로 변경하여 보정 메서드를 활성화하세요.
    return Arrays.asList(
            encoderTicksToInches(parallelEncoder.getCorrectedVelocity()),
            encoderTicksToInches(perpendicularEncoder.getCorrectedVelocity())
    );
}
```

### SampleMecanumDrive에서 로컬라이저(Localizer) 설정하기

로컬라이저 구성을 완료했다면, `SampleMecanumDrive.java` 파일로 돌아갑니다.

대략 131번째 줄에서 "`// TODO: if desired, use setLocalizer() to change the localization method`"라는 주석을 찾을 수 있을 겁니다.

이 주석 아래에 다음 줄을 추가하세요:

```java
/* 약 131번째 줄 - SampleMecanumDrive.java */

// TODO: if desired, use setLocalizer() to change the localization method
// for instance, setLocalizer(new ThreeTrackingWheelLocalizer(...));

setLocalizer(new StandardTrackingWheelLocalizer(hardwareMap));
```

이렇게 로컬라이저 설정을 완료할 수 있습니다.

---

### IMU 제거

세 바퀴 오도메트리(Three-Wheel Odometry) 방식에서는 IMU가 필요하지 않습니다.
따라서 `SampleMecanumDrive`에서 IMU 초기화를 제거하는 것이 좋습니다.
IMU 초기화는 운영 모드(opmode) 초기화 시간을 2~3초 늘릴 수 있으며, 불필요한 시간이 소요될 수 있습니다.

`SampleMecanumDrive.java` 파일에서 다음 코드를 찾아 삭제하세요:

```java
/* 약 134~137번째 줄 - SampleMecanumDrive.java */

// TODO: adjust the names of the following hardware devices to match your configuration
imu = hardwareMap.get(BNO055IMU.class, "imu");
BNO055IMU.Parameters parameters = new BNO055IMU.Parameters();
parameters.angleUnit = BNO055IMU.AngleUnit.RADIANS;
imu.initialize(parameters);
```

그리고 안전을 위해, 아래 메서드들의 반환값을 0으로 변경하세요:

```java {hl_lines=[5, 10]}
/* 약 393~396번째 줄 - SampleMecanumDrive.java */

@Override
public double getRawExternalHeading() {
    return 0;
}

@Override
public Double getExternalHeadingVelocity() {
    return 0.0;
}
```

---

## 조율 (Tuning) - 오도메트리 두 개를 사용하는 경우 (Two-Wheel Odometry)

데드 휠(Dead Wheel)을 사용한 오도메트리의 튜닝은 매우 중요한 과정입니다.
이 과정은 Road Runner뿐만 아니라 FTCLib 또는 직접 개발한 경로 추적 시스템에서도 중요한 역할을 합니다.
이것의 목표는 로컬라이제이션(Localization)의 정확도를 최대화하는 것입니다.

### 바퀴 반지름 조정

{{<callout context="tip" title="팁" icon="outline/rocket">}}
이 과정이 모든 사용자에게 필수는 아니지만, 적용하면 약 1% 더 정확한 위치 추적이 가능합니다.
예를 들어, 100인치 이동 시 1% 오차는 1인치 차이를 줄이는 데 도움이 됩니다.
FTC Skystone 시즌(2019-2020) 동안, 4~5개의 돌을 옮기는 자율 주행 루틴은 100인치 이상을 이동하며,
이런 차이가 큰 영향을 미칠 수 있었습니다.  
{{</callout>}}

{{< callout context="note" title="노트" icon="outline/info-circle" >}}
아래의 과정이 적용된 전체 코드는 [예제 코드](https://gist.github.com/NoahBres/02e83f8317f34d7b627170c2031b2ebf)를 참고하세요.
{{< /callout >}}

1. `TwoWheelTrackingLocalizer.java` 파일을 엽니다.
2. 클래스 내부에 두 개의 변수를 추가하세요:

```java
/* 약 46~47번째 줄 - TwoWheelTrackingLocalizer.java */

public static double X_MULTIPLIER = 1; // X 방향 보정 계수
public static double Y_MULTIPLIER = 1; // Y 방향 보정 계수
```

3. 다음과 같이 `getWheelPositions()`와 `getWheelVelocities()` 함수에 보정 계수를 추가합니다:

```java {hl_lines=[7, 8, 19, 20]}
/* 약 77~97번째 줄 - TwoWheelTrackingLocalizer.java */

@NonNull
@Override
public List<Double> getWheelPositions() {
    return Arrays.asList(
            encoderTicksToInches(parallelEncoder.getCurrentPosition()) * X_MULTIPLIER,
            encoderTicksToInches(perpendicularEncoder.getCurrentPosition()) * Y_MULTIPLIER
    );
}

@NonNull
@Override
public List<Double> getWheelVelocities() {
    // TODO: 인코더 속도가 초당 32767 카운트를 초과할 경우, Encoder.getRawVelocity()를 
    // Encoder.getCorrectedVelocity()로 변경하여 보정 메서드를 활성화하세요.

    return Arrays.asList(
            encoderTicksToInches(parallelEncoder.getRawVelocity()) * X_MULTIPLIER,
            encoderTicksToInches(perpendicularEncoder.getRawVelocity()) * Y_MULTIPLIER
    );
}
```
**참고:** `X_MULTIPLIER`는 수평 엔코더에 작용하는 값입니다. 왜냐하면 x는 로컬 좌표계를 향해 있기 때문입니다 (로봇 공학/항공/등 상황에서 흔히 사용됩니다).

4. 물리적인 튜닝 과정을 진행합니다:
- 로봇이 이동할 수 있는 직선 구간을 준비합니다. 경기장 타일을 직선 형태로 배치하면 좋습니다.
- 로봇을 시작 지점에 두고 앞으로 직진하게 설정합니다.
- `LocalizationTest` opmode를 실행합니다. 단, 컨트롤러는 사용하면 안 됩니다.
- 손으로 로봇을 직선 구간을 따라 천천히 이동시킵니다. 가능한 똑바로 이동하도록 노력하세요. (로봇 옆에 테이프를 붙이고, 테이프와의 간격을 유지하면서 이동시키면 좋습니다.)
- 이동이 끝나면 멈춘 후, 실제 이동한 거리와 텔레메트리(Telemetry)에 기록된 이동 거리를 확인하세요.

5. X 방향 보정 계수 계산:
- `X_MULTIPLIER = 실제 이동 거리 / 텔레메트리 이동 거리`  
  예: 텔레메트리가 89인치를 표시하고 실제로 90인치를 이동했다면, `X_MULTIPLIER`는 `1.01123596`입니다.

6. 동일한 과정을 통해 Y 방향(스트레이프) 보정 계수도 계산하고 설정합니다.

---

### 재확인

1. `LocalizationTest` opmode를 실행합니다.
2. 로봇의 RC에서 `192.168.49.1:8080/dash` 또는 Control Hub에서 `192.168.43.1:8080/dash`로 접속합니다.
3. 컨트롤러로 로봇을 이동시키면서 대시보드 상에서 로봇의 움직임이 실제 움직임과 일치하는지 확인합니다.
4. X 좌표는 앞으로 이동 시 증가해야 하며, Y 좌표는 왼쪽으로 이동 시 증가해야 합니다.

문제가 발생하면 문제 해결(Troubleshooting) 섹션을 참고하세요.

---

## 조율 (Tuning) - 오도메트리 세 개를 사용하는 경우 (Three-Wheel Odometry)

데드 휠의 조율은 매우 중요합니다.
이는 Road Runner뿐만 아니라 FTCLib 또는 직접 제작한 경로 추적 시스템에도 적용됩니다.
정확한 조율은 위치 및 거리 측정에서 발생하는 오류를 최소화하여 자율 주행 시 성능을 크게 향상시킬 수 있습니다.

---

### 바퀴 반지름 조정

{{<callout context="tip" title="팁" icon="outline/rocket">}}
이 단계는 선택 사항이지만 로컬라이제이션 정확도를 약 1% 정도 향상시킬 수 있습니다.
1%가 별것 아닌 것처럼 보일 수 있지만, 100인치 이상의 주행 시 약 1인치 차이를 만들 수 있습니다.
예를 들어 FTC Skystone(2019-2020) 시즌에서 4~5개의 스톤을 운반하는 자율 주행은 100인치 이상 이동해야 했으며,
1인치의 추가 정확도가 큰 차이를 만들었을 수 있습니다.
{{</callout>}}

{{< callout context="note" title="노트" icon="outline/info-circle" >}}
아래의 과정이 적용된 전체 코드는 [예제 코드](https://gist.github.com/NoahBres/9b9710eaa9f9fd23efa30a16de0f610e)를 참고하세요.
{{< /callout >}}

1. **`StandardTrackingWheelLocalizer.java` 파일 열기**  
   해당 파일에서 아래 두 변수를 클래스에 선언합니다:

   ```java
   /* StandardTrackingWheelLocalizer.java 파일의 약 37~38줄 */
   public static double X_MULTIPLIER = 1; // X 방향 보정 계수
   public static double Y_MULTIPLIER = 1; // Y 방향 보정 계수
   ```

2. **보정 계수를 `getWheelPositions()` 및 `getWheelVelocities()` 함수에 추가**  
   아래와 같이 코드를 수정합니다:

   ```java{hl_lines=[15-17,34-36]}
   /* StandardTrackingWheelLocalizer.java 파일 약 67~103줄 */
   @NonNull
   @Override
   public List<Double> getWheelPositions() {
       return Arrays.asList(
           encoderTicksToInches(leftEncoder.getCurrentPosition()) * X_MULTIPLIER,
           encoderTicksToInches(rightEncoder.getCurrentPosition()) * X_MULTIPLIER,
           encoderTicksToInches(frontEncoder.getCurrentPosition()) * Y_MULTIPLIER
       );
   }

   @NonNull
   @Override
   public List<Double> getWheelVelocities() {
       return Arrays.asList(
           encoderTicksToInches(leftEncoder.getCorrectedVelocity()) * X_MULTIPLIER,
           encoderTicksToInches(rightEncoder.getCorrectedVelocity()) * X_MULTIPLIER,
           encoderTicksToInches(frontEncoder.getCorrectedVelocity()) * Y_MULTIPLIER
       );
   }
   ```

   **참고:** X 방향 보정 계수는 X 좌표가 로컬 좌표계에서 앞쪽으로 향하기 때문에 `leftEncoder`와`rightEncoder`에 적용됩니다.

4. 물리적인 튜닝 과정을 진행합니다:
- 로봇이 이동할 수 있는 직선 구간을 준비합니다. 경기장 타일을 직선 형태로 배치하면 좋습니다.
- 로봇을 시작 지점에 두고 앞으로 직진하게 설정합니다.
- `LocalizationTest` opmode를 실행합니다. 단, 컨트롤러는 사용하면 안 됩니다.
- 손으로 로봇을 직선 구간을 따라 천천히 이동시킵니다. 가능한 똑바로 이동하도록 노력하세요. (로봇 옆에 테이프를 붙이고, 테이프와의 간격을 유지하면서 이동시키면 좋습니다.)
- 이동이 끝나면 멈춘 후, 실제 이동한 거리와 텔레메트리(Telemetry)에 기록된 이동 거리를 확인하세요.

5. 보정 계수를 계산합니다.
   보정 계수는 `실제 이동 거리 / 텔레메트리(Telemetry) 이동 거리`로 계산됩니다.  
   예를 들어 텔레메트리가 89인치를 표시하고 실제 측정값이 90인치라면, 보정 계수는 `1.01123596`이 됩니다.

9. 이 과정을 X방향(앞으로 이동)으로 3번 반복해 평균값을 구한 후 `X_MULTIPLIER`에 설정합니다.

10. Y방향(옆으로 이동)에서도 동일한 과정을 반복한 후, 계산된 Y 방향 계수를 `Y_MULTIPLIER`에 설정합니다.

---

### 측면 거리 (Lateral Distance) 조율

측면 거리는 로컬라이제이션에서 방향 계산에 중요한 요소입니다.
이 값을 잘못 설정하면 방향 오차가 누적되어 로컬라이제이션 성능이 크게 저하됩니다.

1. `StandardTrackingWheelLocalizer.java` 파일의 `LATERAL_DISTANCE` 값 확인을 확인합니다.  
   해당 값을 실제 측정한 거리로 설정합니다. 이후 튜닝 과정에서 조율될 예정이니 정확하지 않더라도 괜찮습니다.

2. 로봇의 특정 지점(테이프 표시 등)을 기준으로 360° 회전 전후의 위치를 확인할 수 있도록 준비합니다.

3. `TrackingWheelLateralDistanceTuner` OpMode를 실행합니다.

4. 대시보드를 사용하면 로봇의 상태를 시각적으로 확인할 수 있습니다. RC의 WiFi에 연결한 뒤, 브라우저에서 `192.168.49.1:8080/dash` (RC 전화 사용 시) 또는 `192.168.43.1:8080/dash` (Control Hub 사용 시)로 접속하세요.

5. OpMode를 실행한 후 게임패드의 오른쪽 조이스틱을 사용해 로봇을 **반시계 방향**으로 회전시킵니다.

6. 로봇을 **10회 반시계 방향**으로 회전시키고 완료되면 게임패드의 `Y` 버튼을 눌러 튜닝을 종료합니다. 이후 시작 지점과 동일하게 정렬되었는지 확인합니다.

7. 출력된 `LATERAL_DISTANCE` 값을 `StandardTrackingWheelLocalizer.java`에 입력합니다.

8. 만약 값이 부정확하다면 직접 조정하면서 반복 테스트합니다.

---

## 문제 해결 (Troubleshooting)

### 대시보드와 로봇 움직임 불일치 문제 해결:
1. **대시보드에서 로봇이 실제와 반대로 옆으로 이동할 경우**
- 수직 인코더의 방향을 반대로 설정하세요.

2. **대시보드에서 로봇이 제자리에서 올바르게 회전하지 않을 경우**
- 수직 휠의 위치가 잘못되어 회전 중심에 오프셋이 생긴 것입니다.  
  수직 휠의 위치 조정은 복잡할 수 있으니, 약간의 오프셋은 그대로 두어도 괜찮습니다.  
  이로 인해 로컬라이제이션 정확도는 영향을 받지 않으며, 단지 위치가 약간 이동하는 것뿐입니다.

3. **대시보드에서 로봇이 직진하는 동안 회전할 경우**
- 수평 엔코더 중 하나가 반대로 설정되어 있는 것입니다.

4. **대시보드에서 로봇이 직진 및 스트레이프는 제대로 하지만, 회전 방향이 실제와 반대일 경우**
- 좌측 및 우측 인코더가 서로 바뀌어 연결되어 있는 것입니다.

5. **시간이 지남에 따라 로컬라이제이션 정확도가 떨어질 경우**
- 로봇을 천천히 움직이며 테스트하세요. 가속을 최소화하려고 노력합니다.
  - 느린 속도에서도 로컬라이제이션이 부정확하다면, 설정 값을 다시 확인하세요.  
    헤딩(방향)이 정확한지 확인하고, 스트레이프 및 전진 움직임에서 측정된 거리와 실제 이동 거리가 일치하는지 확인하세요.
- 만약 느린 속도에서는 정확하지만, 빠른 속도에서 정확도가 떨어진다면 이는 하드웨어 문제일 가능성이 높습니다.
  - 가장 흔한 원인은 데드 휠의 마찰력이 부족한 경우입니다. 데드 휠의 스프링 압력을 증가시켜 마찰력을 높이세요.
