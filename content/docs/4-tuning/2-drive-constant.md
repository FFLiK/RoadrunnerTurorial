---
title: "드라이브 상수 설정 (Drive Constant)"
description: "로드러너를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 402
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

{{<callout context="tip" title="팁" icon="outline/rocket">}}
여기에서 편집할 드라이브 상수 파일입니다.

[DriveConstants.java](https://github.com/acmerobotics/road-runner-quickstart/blob/quickstart1/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/DriveConstants.java)

아직 이 파일을 프로젝트에 복사하지 않았다면, 복사해서 사용하세요!
{{</callout>}}

구성기(Configurator)를 사용하면 상수를 빠르게 생성할 수 있습니다. (단 현재 문서에서는 아직 기능을 지원하지 않습니다.)

기능을 이용하기 위해서는 아래 버튼을 클릭한 후 `Configure Me!` 버튼을 눌러주세요.

<a class="btn btn-primary btn-cta rounded-pill btn-lg my-3" href="https://learnroadrunner.com/drive-constants.html#drive-constants" role="button">구성기 사용하러 가기</a>

아래에서는 각 상수가 무엇을 의미하는지 하나씩 살펴보겠습니다.

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
자동 구성을 사용하는 경우, IMU 정보를 직접 파일에 추가해야 합니다. IMU 정보는 [이곳](/drive-constants.html#samplemecanumdrive-imu-velocity)에서 확인하세요.
{{< /callout >}}

```kroki {type=PlantUML}

skinparam ranksep 30
skinparam dpi 125
rectangle "현재 단계" { 
rectangle "<b>드라이브 상수 설정</b>" as s1
}
rectangle "<b>데드 휠 설정</b>" as s2
rectangle "<b>로컬라이제이션 테스트</b>" as s3
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

---

## 회전당 틱 수 (TPR) & 최대 RPM

```java
/* DriveConstants.java 파일의 약 24~25번째 줄 */
public static final double TICKS_PER_REV = 1;
public static final double MAX_RPM = 1;
```

### **`TICKS_PER_REV`**
이 값은 모터 엔코더가 한 바퀴 회전할 때 카운트하는 "틱"의 수를 나타냅니다. 모터의 사양은 제조업체의 사이트에서 확인할 수 있습니다.  
예를 들어, goBILDA의 5202/5203/5204 모터의 경우 [여기](https://www.gobilda.com/yellow-jacket-planetary-gear-motors)에 있는 각 모터의 사양 페이지에서 "Specs" 섹션에 표시된 **Encoder Resolution** 값을 사용해야 합니다.  
다음은 goBILDA가 아닌 몇몇 모터의 사양 리스트입니다. 일부 제조업체는 이 정보를 명확하게 제공하지 않으므로 참고하시기 바랍니다. (goBILDA에 감사드립니다!)

<div class="flex justify-center">

| 모터                          | 회전당 틱 수 | 최대 RPM |
| ----------------------------- | :----------: | -------: |
| REV HD Hex 40:1 Spur          |     1120     |      150 |
| REV HD Hex 20:1 Spur          |      560     |      300 |
| REV HD Hex 20:1 Planetary\*   |    537.6     |    312.5 |
| NeveRest Classic 60           |     1680     |      105 |
| NeveRest Classic 40           |     1120     |      160 |
| NeveRest Orbital 20\*         |    537.6     |      349 |
| NeveRest Orbital 3.7          |    103.6     |     1780 |
| TETRIX TorqueNADO 60:1        |     1440     |      100 |
| TETRIX TorqueNADO 40:1        |      960     |      150 |
| TETRIX TorqueNADO 20:1        |      480     |      480 |

</div>

---

### **`MAX_RPM`**
이 값은 권장 전압(12V)에서 모터가 도달할 수 있는 최대 RPM(분당 회전 수)을 나타냅니다.

\* 이러한 모터는 20:1 비율로 표시되지만, 실제로는 19.2:1의 비율을 가지고 있습니다. 이는 플래니터리 기어 구조로 인해 20:1 스퍼 모터와 속도/틱 값이 다릅니다.

---

## 엔코더 사용 및 모터 속도 PID 설정

```java
/* DriveConstants.java 파일의 약 35~37번째 줄 */
public static final boolean RUN_USING_ENCODER = true;
public static PIDFCoefficients MOTOR_VELO_PID = new PIDFCoefficients(0, 0, 0,
        getMotorVelocityF(MAX_RPM / 60 * TICKS_PER_REV));
```

### **`RUN_USING_ENCODER`**
이 값은 FTC SDK에 내장된 [RunMode](https://gm0.org/en/stable/docs/software/using-the-sdk.html#dc-motor) 중 하나인 `RUN_USING_ENCODER`를 사용할지 여부를 결정합니다. 이를 통해 모터를 "전력" (전압)이 아니라 속도로 제어할 수 있는 온보드 속도 PID를 활용할 수 있습니다. 이 값을 `true`로 설정하면 모든 모터가 속도 제어 모드로 자동 설정됩니다.  
`RUN_USING_ENCODER`는 구동 엔코더를 사용할 때만 사용할 수 있습니다. 구동 엔코더를 사용하지 않는 경우 이 값을 `false`로 설정하세요.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
피드포워드(Fedforward) 튜닝을 사용하는 것이 권장되므로, 구동 엔코더를 사용하더라도 이 값을 `false`로 설정해 두는 것이 좋습니다.
{{</callout>}}

### **`MOTOR_VELO_PID`**
이 값은 사용할 PID 값을 저장합니다. SDK의 기본 PIDF 값은 부하 없이 자유롭게 회전하는 모터를 기준으로 조정되었으며, 이는 구동 트레인에는 적합하지 않습니다.  
따라서 Quickstart는 이 값을 기본값인 0으로 설정하며, 나중에 튜닝을 진행합니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
피드포워드 제어 방식이 권장되므로 이 값을 변경하지 마세요.
{{</callout>}}

---

## 바퀴 반지름 / 기어비 / 트랙 폭

```java
/* DriveConstants.java 파일의 약 47~49번째 줄 */
public static double WHEEL_RADIUS = 2; // 인치 단위
public static double GEAR_RATIO = 1; // 출력(바퀴) 속도 / 입력(모터) 속도
public static double TRACK_WIDTH = 1; // 인치 단위
```

### **`WHEEL_RADIUS`**
구동 트레인의 바퀴 반지름을 나타냅니다. 이 값은 직경이 아니라 **반지름**이어야 합니다.

---

### **`GEAR_RATIO`**
출력(바퀴) 속도를 입력(모터) 속도로 나눈 값입니다.  
기어나 벨트 없이 직구동(direct drive)을 사용하는 경우, `GEAR_RATIO`는 `1`로 설정해야 합니다.  
기어비가 1보다 크면 바퀴가 모터보다 더 빠르게 회전함을 나타내고, 1보다 작으면 바퀴가 모터보다 더 느리게 회전함을 나타냅니다.  
예를 들어, 2019년 v1 goBILDA Strafer Kit에는 1:2 기어가 포함되어 있어 출력 속도가 절반으로 줄어듭니다. 이 경우 기어비는 `1/2` 또는 `0.5`입니다.

{{<callout context="caution" title="주의" icon="outline/alert-triangle">}}
기어비를 분수로 설정할 경우, 반드시 소수값을 사용하세요.

```java
// 이렇게 하면 안 됩니다
public static double GEAR_RATIO = 2 / 3;

// 이렇게 하세요
public static double GEAR_RATIO = 2.0 / 3.0;
```

첫 번째 경우는 정수 나누기(integer division)로 처리되어 값이 0이 된 다음 double 타입으로 변환됩니다.
{{</callout>}}

---

### **`TRACK_WIDTH`**
한쪽 바퀴의 중심에서 평행한 다른 쪽 바퀴의 중심까지의 거리입니다.  
이 값은 대략적인 추정치만으로도 충분하며, 나중에 경험적으로 튜닝할 것입니다.


{{< figure
src="images/drive-constants/wes-bot-edit-half.jpg"
alt="트랙 폭은 한쪽 바퀴의 중심에서 다른 쪽 바퀴의 중심까지의 거리입니다."
caption="2019/20 Skystone 시즌의 3658 Bosons 로봇"
>}}
>

---

## **kV/kA/kStatic**

```java
/* DriveConstants.java 파일의 약 57~59번째 줄 */
public static double kV = 1.0 / rpmToVelocity(MAX_RPM);
public static double kA = 0;
public static double kStatic = 0;
```

이 값들은 구동 모터의 피드포워드 게인을 나타냅니다.  
피드포워드 방식을 선택한 경우, 나중에 값을 튜닝할 수 있습니다. 현재는 기본값으로 두세요.

---

### **`kV`**
- 단위: 볼트 \* 초 / 미터
- 이론적으로, `kV`는 12볼트를 모터의 이론적 자유 속도로 나눈 값입니다.

---

### **`kA`**
- 단위: 볼트 \* 초² / 미터

---

### **`kStatic`**
- 단위: 볼트

---

모터 모델에 대한 추가 정보는 [_Controls Engineering in FRC_ by Tyler Veness](https://file.tavsys.net/control/controls-engineering-in-frc.pdf)에서 확인할 수 있습니다.  
이 상수들의 효과는 나중에 시연을 통해 설명할 예정입니다.

---

## 기본 제한값 설정(Base Constraints)

```java
/* DriveConstants.java 파일의 약 68~71번째 줄 */
public static double MAX_VEL = 30;
public static double MAX_ACCEL = 30;
public static double MAX_ANG_VEL = Math.toRadians(180);
public static double MAX_ANG_ACCEL = Math.toRadians(180);
```

---

### **`MAX_VEL`**
로봇이 달성할 수 있는 **최대 속도**를 정의합니다.  
이 값은 로봇이 가속하여 도달할 수 있는 가장 빠른 속도를 나타냅니다. 기본값은 `30인치/초`입니다. 이론적인 최대 속도는 아래의 공식을 통해 계산할 수 있습니다:

```math
$$
[\text{Velocity}_{\text{max}}] = \frac{\text{RPM}_{\text{max}}}{60} \times \text{Gear Ratio} \times \text{Wheel Radius} \times 2\pi
$$
```

로봇의 최대 속도 제약은 모터 최대 속도의 80%를 초과하지 않도록 설정하는 것이 좋습니다. 배터리 전압 감소, 무게 등 여러 요인으로 인해 이론적인 최대 속도에 도달하기 어려울 수 있습니다.  
최대 속도를 더 높게 설정할 수도 있지만, 로봇이 해당 속도에 도달하지 못하면 경로 추적 성능이 저하될 수 있습니다.

`MaxVelocityTuner` 오프모드를 사용하여 최대 속도를 실험적으로 정의할 수 있습니다. 그러나, `MAX_VEL`은 `MaxVelocityTuner` 출력값의 **90~95%**로 설정하는 것이 권장됩니다.

---

### **`MAX_ACCEL`**
로봇이 가속할 수 있는 **최대 가속도**를 정의합니다.  
이 값은 속도가 얼마나 빠르게 증가하는지를 나타냅니다. 기본값은 `30인치/초²`입니다.  
처음에는 이 값을 최대 속도(`MAX_VEL`)와 동일하게 설정하는 것이 권장되며, 이는 대략적인 기준입니다.  
실험적으로 값을 결정하려면 가속도를 점진적으로 높이면서 경로 추적 성능이 저하되는 시점을 찾으면 됩니다.  
이 작업은 PID 튜닝 후에 수행하는 것이 더 쉽습니다.

---

### **`MAX_ANG_VEL`**
로봇이 회전할 수 있는 **최대 각속도**를 정의합니다.  
이 값은 로봇이 가장 빠르게 회전할 수 있는 속도를 나타내며 기본값은 `180°/초`입니다.  
최대 각속도는 최대 접선 속도(`MAX_VEL`)를 트랙 폭(`TRACK_WIDTH`)으로 나누어 계산할 수 있습니다.  
그러나 실험적으로 값을 결정하거나 기본값을 사용하는 것이 좋습니다.  
이론적인 최대 각속도와 측정값은 잘 일치하지 않을 수 있습니다.

---

### **`MAX_ANG_ACCEL`**
로봇이 회전할 수 있는 **최대 각가속도**를 정의합니다.  
이 값은 로봇의 각속도가 얼마나 빠르게 증가할 수 있는지를 나타내며 기본값은 `180°/초²`입니다.  
실험적으로 값을 찾을 수 있지만, 이를 정확히 측정하는 것은 어려울 수 있으므로 기본값을 사용하는 것이 좋습니다.

---

## SampleMecanumDrive - 하드웨어 ID

`SampleMecanumDrive.java` 파일을 엽니다.

```java
/* SampleMecanumDrive.java 파일의 약 102~105번째 줄 */
leftFront = hardwareMap.get(DcMotorEx.class, "leftFront");
leftRear = hardwareMap.get(DcMotorEx.class, "leftRear");
rightRear = hardwareMap.get(DcMotorEx.class, "rightRear");
rightFront = hardwareMap.get(DcMotorEx.class, "rightFront");
```

모터 ID가 Rev Hub 설정에서의 ID와 일치하는지 확인하세요.

---

### **IMU 방향 설정**

드라이브 엔코더 로컬라이제이션을 사용 중이라면, `DriveConstants.java` 파일에서 `RevHubOrientationOnRobot`을 찾아 설정하거나, 존재하지 않는다면 아래 코드와 같이 추가하세요.  
자세한 설정 방법은 [FTC 문서](https://ftc-docs.firstinspires.org/en/latest/programming_resources/imu/imu.html)를 참조하세요.

{{< figure
src="images/drive-constants/control-hub-orientation-example.png"
alt="Control Hub 방향 예제"
caption="Control Hub 방향 예제"
>}}

```java
/* DriveConstants.java 파일의 약 76번째 줄 */
public static RevHubOrientationOnRobot.LogoFacingDirection LOGO_FACING_DIR =
        RevHubOrientationOnRobot.LogoFacingDirection.UP;
public static RevHubOrientationOnRobot.UsbFacingDirection USB_FACING_DIR =
        RevHubOrientationOnRobot.UsbFacingDirection.FORWARD;
```

---

### **IMU 속도 설정**

드라이브 엔코더 로컬라이제이션이나 두 개의 데드 휠을 사용하는 경우, `SampleMecanumDrive.java` 파일의 맨 아래에서 `getExternalHeadingVelocity` 함수를 찾으세요.  
이 함수가 IMU가 로봇의 축을 따라 회전하는 축(예: z 축)에 대해 올바른 값을 반환하는지 확인하세요.  
REV Hub가 평평하게 장착된 경우 로봇은 z축을 따라 회전하며, 다른 방향으로 장착된 경우 x 또는 y 축을 따라 회전할 수 있습니다.

```java
/* SampleMecanumDrive.java 파일의 약 296~299번째 줄 */
@Override
public Double getExternalHeadingVelocity() {
    return (double) imu.getRobotAngularVelocity(AngleUnit.RADIANS).zRotationRate;
}
```

{{< figure
src="images/drive-constants/control-hub-axes-diagram.png"
alt="Control Hub 축 다이어그램"
caption="Control Hub 축 다이어그램"
>}}

---

### **모터 방향 설정**

`SampleMecanumDrive.java` 파일의 약 125~127번째 줄로 이동합니다. "`// TODO: reverse any motors using DcMotor.setDirection()`"라는 주석을 찾으세요.  
그 아래에 로봇의 한쪽 모터 방향을 반대로 설정하세요.  
테스트 중 로봇이 원을 그리며 움직이거나, 방향이 반대이거나, 경로를 제대로 따르지 않는다면 여기로 돌아와 문제를 수정하세요.

```java
/* SampleMecanumDrive.java 파일의 약 125~127번째 줄 */

// TODO: reverse any motors using DcMotor.setDirection()
rightFront.setDirection(DcMotorSimple.Direction.REVERSE); // 필요 시 추가
rightRear.setDirection(DcMotorSimple.Direction.REVERSE); // 필요 시 추가
```

모터 설정 문제를 해결하려면 [Motor Direction Debugger opmode](https://github.com/acmerobotics/road-runner-quickstart/blob/quickstart1/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/opmode/MotorDirectionDebugger.java)를 참조하세요.  
`@Disabled` 주석을 제거하고, opmode의 지침을 따르세요.  
또한, 아래의 goBILDA 메카넘 휠 방향 차트를 참고하여 문제를 진단하고 적절히 수정하세요.

{{< figure
src="images/drive-constants/gobilda-mecanum-chart.png"
alt="goBILDA 메카넘 휠 방향 차트"
caption="goBILDA 메카넘 휠 방향 차트"
>}}