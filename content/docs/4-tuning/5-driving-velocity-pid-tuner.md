---
title: "주행 속도 PID 조율 (Driving Velocity PID Tuner)"
description: "Road Runner를 사용하기 전 로봇에 맞게 시스템을 조율하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 405
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
이 페이지는 **특별한 목적이 있는 경우를 제외하고는 사용하지 않는 것이 좋습니다**.
주행 속도 PID 조율 (Drive Velocity PID Tuning)은 ***로봇에서 사용하지 않는 것을 권장합니다***. 
대신 [Feedforward 제어](../주행-피드포워드-조율-driving-feedforward-tuner)를 사용하세요.
{{< /callout >}}

---

## 주행 속도 PID 조율 (Drive Velocity PID Tuning)를 사용하지 않는 이유
주행 속도 PID 조율은 SDK의 내부 PID 컨트롤러를 사용하므로, 튜닝 과정이 매우 까다롭습니다. 
이 방법으로도 좋은 결과를 얻을 수 있지만, [Feedforward](../주행-피드포워드-조율-driving-feedforward-tuner)를 사용하는 것이 
훨씬 더 간단하며 동일한 정확도를 얻을 수 있습니다. 
DriveVelocityControl은 특히 방향 및 팔로워 PID를 통합할 때 수정할 수 없는 진동을 유발할 수 있으며, 
문제가 없더라도 튜닝에 더 많은 시간이 걸리는 경우가 많습니다.

---

## DriveVelocityPID 튜닝

{{< callout context="danger" title="경고" icon="outline/alert-octagon" >}}
이 섹션은 엔코더를 사용하지 않도록 선택한 경우 생략해야 합니다.
{{< /callout >}}

```kroki {type=PlantUML}

skinparam ranksep 30
skinparam dpi 125

rectangle "<b>데드 휠 설정</b>" as s2
rectangle "<b>로컬라이제이션 테스트</b>" as s3
rectangle "현재 단계" {
rectangle "엔코더 사용" { 
rectangle "<b>주행 속도 PID 조율</b>" as s4_y
}
}
rectangle "엔코더 미사용" { 
rectangle "<b>주행 피드포워드 조율</b>" as s4_n
}
rectangle "<b>직진 테스트</b>" as s5

(s2) --> (s3)
(s3) --> (s4_y)
(s3) --> (s4_n)
(s4_y) --> (s5)
(s4_n) --> (s5)

```
현재 단계는 **주행 피드포워드 조율** 단계입니다.

속도 PID 튜닝은 Road Runner에서 가장 까다로운 작업 중 하나일 수 있습니다. 
이는 정확한 경로 추적을 위해 필요합니다. 
하지만 PID 컨트롤러의 작동 원리에 대한 직관과 수행 방법을 이해하면 이러한 좌절감을 완화하고 
과정을 더 원활하게 진행할 수 있습니다.

{{< callout context="tip" title="팁" icon="outline/rocket" >}}
- Xbox 및 Logitech 게임패드에서는 **X 버튼**을, PS4 DualShock에서는 **□ 버튼**을 눌러 튜닝 과정을 일시 중지하고 드라이버 제어로 전환할 수 있습니다.
- Xbox 및 Logitech 게임패드에서는 **A 버튼**을, PS4 DualShock에서는 **X 버튼**을 눌러 튜너에게 제어권을 다시 넘길 수 있습니다.
- 로봇이 경로를 벗어나면 드라이버 제어로 전환하여 로봇을 초기 위치로 되돌리세요.  
{{< /callout >}}

---

## 조율 과정 (Tuning)

1. **조율을 시작하기 전**
  - RC 폰을 시작하고 Dashboard를 엽니다.
  - RC 폰의 Wi-Fi 네트워크에 연결합니다. 네트워크 비밀번호는 `Program and Manage` 메뉴에서 확인할 수 있습니다.

2. **Dashboard 접속**
  - RC 폰을 사용하는 경우, 브라우저에서 `192.168.49.1:8080/dash`로 이동하세요.
  - Control Hub를 사용하는 경우, `192.168.43.1:8080/dash`로 이동하세요.

3. **`MaxVelocityTuner` 실행**
  - Velocity PID 튜닝을 시작하기 전에 `MaxVelocityTuner`를 실행해 경험적 `kF` 값과 최대 속도를 측정합니다.
  - `MaxVelocityTuner`는 지정된 `RUNTIME` 동안 최대 속도로 실행됩니다. 기본값은 **2초**입니다. 충분한 공간이 확보되었는지 확인하세요. `RUNTIME`은 코드나 Dashboard에서 조정할 수 있습니다.

4. **`MaxVelocityTuner` 결과 확인**
  - 실행 후 "Max Velocity" 값과 "Voltage Compensated kF" 값이 출력됩니다.
    - "Max Velocity"는 로봇이 최대 부하와 배터리 수준에서 이동할 수 있는 최대 속도입니다. 이 값을 `DriveConstants`의 `MAX_VEL`로 설정하세요. 약간의 여유를 두기 위해 90-95% 값을 추천합니다.
    - "Voltage Compensated kF" 값을 기록하세요.
  - 만약 로봇이 원을 그리며 회전한다면, 드라이브 트레인의 모터 방향 설정이 잘못되었습니다. [여기](../드라이브-상수-설정-drive-constant/)를 참조하세요.

5. **Dashboard 설정**
  - Dashboard 오른쪽의 `DriveConstants` 드롭다운에서 `MOTOR_VELO_PID`를 열고 `f` 필드에 "Voltage Compensated kF" 값을 입력합니다.

6. **`DriveVelocityPIDTuner` 실행**
  - 로봇이 지정된 거리(기본값: 72인치)를 반복적으로 직선 이동합니다.
  - 충분한 공간(최소 72인치 + 1피트)을 확보하세요. 공간이 부족하다면 Dashboard 또는 파일에서 `DISTANCE` 값을 조정하세요.
  - 로봇이 직선에서 벗어나더라도 걱정하지 마세요. 나중에 헤딩 PID가 이를 보정합니다.

{{< figure
src="images/drive-velocity-pid-tuning/example-dashboard-half.jpg"
alt="브라우저에서 FTC 대시보드를 보여주는 이미지"
caption="예제 대시보드 (그래프 내용은 무시하세요. 이는 페이지 레이아웃의 예시일 뿐입니다.)"
>}}

7. **그래프 설정**
  - 프로그램 실행 후 Dashboard에서 그래프 버튼을 클릭하세요. 그래프가 표시되지 않고 체크박스가 나타나면 `targetVelocity`와 `velocity0` 체크박스를 활성화하세요.

8. **`DISTANCE` 조정**
  - 그래프의 `targetVelocity` 선이 평평한 구간(Plateau)을 가지도록 `DISTANCE` 값을 조정하세요.

9. **PIDF 값 튜닝**
  - `velocity0` 그래프가 `targetVelocity` 선과 일치하도록 PIDF 값을 조정합니다.

10. **튜닝 절차**
  1. 모든 PID 값을 0으로 설정하고, `f`는 `MaxVelocityTuner`에서 얻은 값으로 설정합니다.
  2. `velocity0`이 `targetVelocity`의 꼭대기에 도달하도록 `kF` 값을 조정합니다.
  3. `p` 값을 서서히 증가시켜 그래프의 경사(slope)가 `targetVelocity`와 일치하도록 합니다.
  4. `d` 값을 소량 조정하여 진동을 줄입니다.
  5. `i` 값은 사용하지 않는 것이 좋습니다. 필요하다면 `f` 값을 증가시키세요.
  6. 튜닝 후 Dashboard에서 설정한 값을 `DriveConstants.java` 파일에 복사하세요. Dashboard 설정은 임시로 저장되므로, 이를 잊지 마세요.

11. **추가 팁**
  - 진동을 최소화하는 것을 우선시하세요.
  - 완벽한 튜닝은 불가능하므로, 일정 수준의 편차를 허용하세요.
  - 튜닝 시뮬레이터를 활용해 각 PIDF 값이 동작에 미치는 영향을 확인하세요.

{{< callout context="tip" title="팁" icon="outline/rocket" >}}
Velocity PID 컨트롤러에서는 `kD`가 반드시 필요한 것은 아니지만, FTC 로봇에서는 피드포워드와 모터 컨트롤러 특성 때문에 유용할 수 있습니다.  
`i` 값을 사용하는 대신 `f` 값을 조정하는 것이 더 효과적입니다.  
{{< /callout >}}

---

## 문제 해결 (Troubleshooting)

1. **`MaxVelocityTuner`가 뒤로 움직이는 경우**
    - 휠의 방향이 올바르게 설정되었는지 확인하세요. 디버깅 시 [goBILDA 메카넘 차트](/drive-constants.html#samplemecanumdrive-motor-direction)를 참고하세요.

2. **속도 선 중 하나가 `targetVelocity`와 반대로 움직이는 경우**
    - 모터의 극성이 잘못되었습니다. 인코더가 모터의 실제 회전 방향과 다르게 읽고 있습니다.
        - 모터의 빨간색과 검은색 케이블을 교체하세요.
        - 또는 `SampleMecanumDrive`에서 인코더 값을 -1로 곱하세요.

3. **`StraightTest` 또는 `DriveVelocityPID`가 계속 과도하게 움직이고 `DriveConstants.java` 변수 조정이 효과가 없는 경우**
    - `DriveConstants.java`에서 `MAX_VEL` 값을 낮춰보세요. 문제를 확인하기 위해 처음에는 아주 낮은 값으로 설정하세요.

4. **기타 모터 방향 문제**
    - [모터 방향 설정](/drive-constants.html#samplemecanumdrive-motor-direction)을 참조하세요.
    - 모터 설정 디버깅에 어려움이 있다면 [Motor Direction Debugger opmode](https://github.com/acmerobotics/road-runner-quickstart/blob/quickstart1/TeamCode/src/main/java/org/firstinspires/ftc/teamcode/drive/opmode/MotorDirectionDebugger.java)를 사용하세요.
        - 파일에서 `41`번째 줄의 `@Disabled`를 제거한 후, opmode 주석에 따라 실행하세요.
        - 각 모터를 하나씩 실행하여 문제를 진단하고 적절히 수정하세요.

---

## PID 튜닝 시뮬레이터

현재 이 웹사이트에서는 시뮬레이터를 직접적으로 지원하지 않습니다.
대신, 아래 버튼을 클릭하여 시뮬레이터를 이용해볼 수 있습니다.

<a class="btn btn-primary btn-cta rounded-pill btn-lg my-3" href="https://learnroadrunner.com/drive-velocity-pid-tuning.html#pid-tuning-simulator" role="button">시뮬레이션 바로가기</a>

**튜닝 예제**
- 게인 값을 조정하며 그래프가 어떻게 변하는지 확인하세요.
- 이 그래프는 실제 로봇 튜닝 시와 매우 유사하므로 사전에 이해를 돕는 데 유용합니다.
- 새 게인 값을 입력한 후 **Enter 키**를 눌러 적용하세요.

{{< callout context="tip" title="팁" icon="outline/rocket" >}} 
이 시뮬레이터는 매우 기본적인 수준의 "시뮬레이터"입니다.
- Rev Hub 모터 컨트롤러를 정확히 시뮬레이션하지는 않으며, 단순한 DC 모터 모델을 기반으로 동작합니다.
- 실제 튜닝과 다를 수 있지만 기본적인 동작 원리를 이해하는 데 유용합니다.
- 시뮬레이터의 `kF` 값은 매우 효과적이지만, 실제 로봇에서는 Rev Hub 모터 컨트롤러 특성 때문에 효과가 덜할 수 있습니다.
- 버그가 있을 수 있으니, 문제가 발생하면 "Reset" 버튼을 사용하세요.  
{{< /callout >}}

---

## PID 컨트롤러 학습 자료

**추천 영상**  
PID 컨트롤러의 직관적인 이해를 돕는 몇 가지 좋은 영상입니다.
- 첫 번째 영상은 초반 3-4분을 건너뛰고, 각 게인의 동작을 설명하는 후반부를 참고하세요.
- 두 번째와 세 번째 영상도 게인의 동작을 설명합니다.

1. [PID Control Explained](https://www.youtube.com/watch?v=XfAt6hNV8XM)
2. [Understanding PID Control](https://www.youtube.com/watch?v=6OH-wOsVVjg)
3. [PID Tuning Basics](https://www.youtube.com/watch?v=0vqWyramGy8)

**추가 읽기 자료**
- [Intro to Control: Part One - PID](https://blog.wesleyac.com/posts/intro-to-control-part-one-pid)

---

## 임시 비디오
편집된 영상이 준비되기 전 임시로 제공되는 비디오입니다. 화질은 좋지 않지만 도움이 될 수 있습니다.

<div class="flex justify-center">
   <iframe width="560" height="315" src="https://www.youtube.com/embed/2UkeLsQuCOw" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>