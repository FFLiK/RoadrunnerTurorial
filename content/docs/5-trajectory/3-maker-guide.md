---
title: "마커 (Maker) 가이드"
description: "로드러너를 사용해 자율 주행 경로를 설정하는 방법을 다룹니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 503
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

Trajectory를 다룰 줄 알게 되었다면, 이제 로봇 대회에서 승리하기 위해 한 단계 더 나아가야 합니다. 
물론 정확한 자율 주행만으로도 높은 점수에 가까워질 수 있지만, 
단순히 로봇이 주행하는 것만으로는 충분하지 않습니다. 
모터를 켜고, 서보를 작동시키고, 다양한 요소와 상호작용해야 합니다. 그렇다면 이는 어떻게 구현할 수 있을까요?

Road Runner는 **마커(Marker)** 라는 편리한 기능을 제공합니다. 
이를 통해 경로를 따라 동작을 실행할 수 있습니다.

---

## 기본 마커 예제

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java{hl_lines=[3-6]}
drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))
  .addDisplacementMarker(() -> {
    // 여기에 동작을 실행하세요!
    // 서보를 내리거나 모터를 시작하는 등
  })
  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .build();
```
{{< /tab >}}
{{< tab "Kotlin" >}}
```kotlin{hl_lines=[3-6]}
drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))
  .addDisplacementMarker {
    // 여기에 동작을 실행하세요!
    // 서보를 내리거나 모터를 시작하는 등
  }
  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .build()
```
{{< /tab >}}
{{< /tabs >}}

---

## 마커의 종류

마커에는 세 가지 주요 유형이 있습니다:
1. **Temporal Markers** (시간 기반 마커)
2. **Spatial Markers** (공간 기반 마커)
3. **Displacement Markers** (변위 기반 마커)

---

### 시간 기반 마커 (Temporal Markers)

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java{hl_lines=[4-8]}
drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))

  .addTemporalMarker(2, () -> {
      // 이 마커는 Trajectory 시작 후 2초 뒤에 실행됩니다.
      // 여기에 동작을 실행하세요!
  })

  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .build();
```
{{< /tab >}}
{{< tab "Kotlin" >}}
```kotlin{hl_lines=[4-8]}
drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))

  .addTemporalMarker(2.0) {
      // 이 마커는 Trajectory 시작 후 2초 뒤에 실행됩니다.
      // 여기에 동작을 실행하세요!
  }

  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .build()
```
{{< /tab >}}
{{< /tabs >}}

**Temporal Marker**는 경로를 따라 **시간**을 기준으로 동작을 실행합니다.  
**중요:** Temporal Marker는 "전역(Global)"으로 평가됩니다. 
즉, 마커가 경로 전체를 빌드한 후 평가되기 때문에 `addTemporalMarker` 호출 순서는 중요하지 않습니다.

{{< tabs "create-new-site" >}}
{{< tab "java" >}}
```java{hl_lines=[6,13]}
// 아래 두 트라젝토리는 실행 결과가 동일합니다.
// 마커 호출 순서와 관계없이 2초 뒤에 실행됩니다.

drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))
  .addTemporalMarker(2, () -> {})
  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .build();

drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))
  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .addTemporalMarker(2, () -> {})
  .build();
```
{{< /tab >}}
{{< tab "Kotlin" >}}
```kotlin{hl_lines=[6,13]}
// 아래 두 트라젝토리는 실행 결과가 동일합니다.
// 마커 호출 순서와 관계없이 2초 뒤에 실행됩니다.

drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))
  .addTemporalMarker(2.0) { }
  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .build()

drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))
  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .addTemporalMarker(2.0) { }
  .build()
```
{{< /tab >}}
{{< /tabs >}}

---

### 변위 기반 마커 (Displacement Markers)

변위 기반 마커는 두 가지 방식으로 사용할 수 있습니다.

#### 1. 인라인 변위 마커 (Inline Displacement Marker)

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java{hl_lines=[4-8]}
drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))

  .addDisplacementMarker(() -> {
      // 이 마커는 첫 번째 splineTo() 이후 실행됩니다.
      // 여기에 동작을 실행하세요!
  })

  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .build();
```
{{< /tab >}}
{{< tab "kotlin" >}}
```kotlin{hl_lines=[4-8]}
drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))

  .addDisplacementMarker {
      // 이 마커는 첫 번째 splineTo() 이후 실행됩니다.
      // 여기에 동작을 실행하세요!
  }

  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .build()
```
{{< /tab >}}
{{< /tabs >}}

**특징:**
- 인라인 마커는 순서가 중요합니다.
- 이전 명령이 완료된 후 바로 실행됩니다.
- 특정 트라젝토리 세그먼트 이후 빠르게 동작을 실행하려면 유용합니다.

---

#### 2. 전역 변위 마커 (Global Displacement Marker)

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java{hl_lines=[4-8]}
drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))

  .addDisplacementMarker(20, () -> {
      // 이 마커는 경로에서 20인치 이동한 후 실행됩니다.
      // 여기에 동작을 실행하세요!
  })

  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .build();
```
{{< /tab >}}
{{< tab "kotlin" >}}
```kotlin{hl_lines=[4-8]}
drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))

  .addDisplacementMarker(20.0) {
      // 이 마커는 경로에서 20인치 이동한 후 실행됩니다.
      // 여기에 동작을 실행하세요!
  }

  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .build()
```
{{< /tab >}}
{{< /tabs >}}

**특징:**
- 변위를 기준으로 동작을 실행합니다.
- Temporal Marker와 마찬가지로 "전역(Global)"으로 평가됩니다.
- 호출 순서와 관계없이 지정된 변위 이후 실행됩니다.

{{< tabs "create-new-site" >}}
{{< tab "java" >}}
```java{hl_lines=[6,13]}
// 아래 두 트라젝토리는 실행 결과가 동일합니다.
// 마커 호출 순서와 관계없이 20인치 이동 후 실행됩니다.

drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))
  .addDisplacementMarker(20, () -> {})
  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .build();

drive.trajectoryBuilder(new Pose2d())
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))
  .splineTo(new Vector2d(72, 0), Math.toRadians(0))
  .addDisplacementMarker(20, () -> {})
  .build();
```
{{< /tab >}}
{{< tab "kotlin" >}}
```kotlin{hl_lines=[6,13]}
// 아래 두 트라젝토리는 실행 결과가 동일합니다.
// 마커 호출 순서와 관계없이 20인치 이동 후 실행됩니다.

drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))
  .addDisplacementMarker(20.0) { }
  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .build()

drive.trajectoryBuilder(Pose2d())
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))
  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))
  .addDisplacementMarker(20.0) { }
  .build()
```
{{< /tab >}}
{{< /tabs >}}

### 공간 마커 (Spatial Marker)
{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java{hl_lines=[4-8]}  
drive.trajectoryBuilder(new Pose2d())  
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))  

  .addSpatialMarker(new Vector2d(20, 20), () -> {  
      // 이 마커는 (20, 20) 좌표에 가장 가까운 지점에서 실행됩니다.  

      // 여기에 동작을 추가하세요!  
  })  

  .splineTo(new Vector2d(72, 0), Math.toRadians(0))  
  .build();  
```  

{{< /tab >}}
{{< tab "Kotlin" >}}
```kotlin{hl_lines=[4-8]}  
drive.trajectoryBuilder(Pose2d())  
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))  

  .addSpatialMarker(Vector2d(20.0, 20.0)) {  
      // 이 마커는 (20, 20) 좌표에 가장 가까운 지점에서 실행됩니다.  

      // 여기에 동작을 추가하세요!  
  }  

  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))  
  .build()  
```  
{{< /tab >}} 
{{< /tabs >}}

공간 마커는 지정된 좌표를 기준으로 동작을 실행할 수 있도록 합니다. 
하지만 Road Runner는 사용자가 지정한 좌표를 경로에 투영(projection)하기 때문에 동작이 실행되는 위치가 약간 예측 불가능할 수 있습니다. 
즉, 경로 상에서 지정된 좌표에 가장 가까운 지점에서 실행됩니다. 
개인적으로는 공간 마커가 명확하지 않기 때문에 사용하는 것을 권장하지 않습니다.

{{< figure
src="images/marker-overview/spatial-projection.jpg"
alt="경로에 투영된 공간 마커를 나타내는 이미"
caption="경로에 투영된 공간 마커"
>}} 

---

## 유의해야 할 사항

### 모든 마커는 시간 기반 마커

변위 마커(displacement marker)와 공간 마커(spatial marker)는 
경로가 빌드될 때 내부적으로 시간 기반 마커로 변환됩니다. 
이는 Road Runner가 이러한 마커를 타이머를 기반으로 실행한다는 것을 의미합니다. 
경로 추적(path following)이 정확하다면 이러한 방식은 꽤 신뢰할 만합니다. 
그러나 경로 추적이 부정확하다면 마커 호출이 지연될 수 있습니다.

따라서 현재로서는 시간 기반이 아닌 경로 기반 마커는 존재하지 않습니다.

### 마커에서 `sleep()`을 사용하지 마세요!

기본 Linear OpMode에서 `sleep()`을 사용해 동작 간 대기 시간을 설정했을 수 있습니다. 
예를 들어, 서보를 떨어뜨리는데 0.5초가 걸리고, 다음 동작을 0.5초 후에 호출하고 싶다고 가정해 봅시다. 
또는 모터를 1초 동안 켜고 이후 꺼야 할 경우, 다음과 같이 작성할 수 있습니다:

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java  
// 이런 방식은 사용하지 마세요!  
drive.trajectoryBuilder(new Pose2d())  
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))  
  .splineTo(new Vector2d(72, 0), Math.toRadians(0))  
  .addDisplacementMarker(10, () -> {  
    // 모터를 켜고 80% 출력으로 설정  
    motor1.setPower(0.8);  

    // 1500ms 대기  
    sleep(1500);  

    // 모터 끄기  
    motor1.setPower(0);  
  })  
  .build();  
```  

{{< /tab >}}
{{< tab "Kotlin" >}}

```kotlin  
// 이런 방식은 사용하지 마세요!  
drive.trajectoryBuilder(Pose2d())  
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))  
  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))  
  .addDisplacementMarker(10.0) {  
    // 모터를 켜고 80% 출력으로 설정  
    motor1.power = 0.8  

    // 1500ms 대기  
    sleep(1500.0)  

    // 모터 끄기  
    motor1.power = 0.0  
  }  
  .build()  
```  

{{< /tab >}}
{{< /tabs >}}

이러한 방식은 좋지 않습니다. 
`sleep()` 함수는 전체 스레드를 멈추게 합니다. 
즉, 코드가 1.5초 동안 "정지"됩니다. 
이 동안 로봇은 속도나 드라이브 트레인을 백그라운드에서 조정할 수 없으며 모션 프로파일이 무너질 수 있습니다.

대신, 두 개의 시간 기반 마커를 체이닝하거나 여러 변위 마커를 세밀하게 조정하는 방식을 권장합니다.

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java  
drive.trajectoryBuilder(new Pose2d())  
  .splineTo(new Vector2d(36, 36), Math.toRadians(0))  
  .splineTo(new Vector2d(72, 0), Math.toRadians(0))  
  .addTemporalMarker(1, () -> {  
    // 경로 시작 1초 후 실행  

    // 모터를 켜고 80% 출력으로 설정  
    motor1.setPower(0.8);  
  })  
  .addTemporalMarker(2.5, () -> {  
    // 경로 시작 2.5초 후 실행 (이전 마커에서 1.5초 후)  

    // 모터 끄기  
    motor1.setPower(0);  
  })  
  .build();  
```  

{{< /tab >}}
{{< tab "Kotlin" >}}

```kotlin  
drive.trajectoryBuilder(Pose2d())  
  .splineTo(Vector2d(36.0, 36.0), Math.toRadians(0.0))  
  .splineTo(Vector2d(72.0, 0.0), Math.toRadians(0.0))  
  .addTemporalMarker(1.0) {  
    // 경로 시작 1초 후 실행  

    // 모터를 켜고 80% 출력으로 설정  
    motor1.power = 0.8  
  }  
  .addTemporalMarker(2.5) {  
    // 경로 시작 2.5초 후 실행 (이전 마커에서 1.5초 후)  

    // 모터 끄기  
    motor1.power = 0  
  }  
  .build()  
```
{{< /tab >}}
{{< /tabs >}}

---

## 고급 설정

### 시간 기반 마커 : 람다 매개변수

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}

```java  
addTemporalMarker(pathTime -> pathTime * 0.3, () -> {  
  // 경로의 30% 지점에서 실행됩니다.  
})  
```
{{< /tab >}}
{{< tab "Kotlin" >}}
```kotlin  
addTemporalMarker({ pathTime -> pathTime * 0.3 }) {  
  // 경로의 30% 지점에서 실행됩니다.  
}  
```  
{{< /tab >}}
{{< /tabs >}}

첫 번째 매개변수는 경로의 전체 시간을 제공하는 콜백 함수입니다. 
이를 통해 경로에 대한 명확한 제어를 할 수 있습니다. 
원하는 값을 반환할 수 있으며, 나머지 시간 기반 마커는 이 함수에 기반을 둡니다. 
이 기능을 사용하는 것은 권장되지 않지만, 필요할 경우 사용할 수 있도록 제공됩니다.

---

### 변위 기반 마커 : 비율(Scale)과 오프셋(Offset)

```java  
addDisplacementMarker(scale: Double, offset: Double, callback: MarkerCallback)  
```

변위 기반 마커는 비율(Scale)과 오프셋(Offset)을 사용할 수 있습니다.
- **비율(Scale)**: 0에서 1까지의 값을 사용하며, 이는 경로 전체 변위의 비율을 나타냅니다.
- **오프셋(Offset)**: 미세한 타이밍 조정을 위해 사용할 임의의 숫자입니다.

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}

```java  
.addDisplacementMarker(0.5, 2.1, () -> {  
  // 이 예제는 경로의 50% 지점에서 실행되며, 추가로 2.1 인치 이동 후 실행됩니다.  
  // 오프셋은 0으로 둘 수 있지만, 타이밍을 미세하게 조정하는 데 유용합니다.  
})  
```  
{{< /tab >}}
{{< tab "Kotlin" >}}
```kotlin  
.addDisplacementMarker(0.5, 2.1) {  
  // 이 예제는 경로의 50% 지점에서 실행되며, 추가로 2.1 인치 이동 후 실행됩니다.  
  // 오프셋은 0으로 둘 수 있지만, 타이밍을 미세하게 조정하는 데 유용합니다.  
}  
```  
{{< /tab >}}
{{< /tabs >}}

---

### 변위 기반 마커 : 람다 매개변수

{{< tabs "create-new-site" >}}
{{< tab "Java" >}}
```java  
.addDisplacementMarker(pathLength -> pathLength * 0.3, () -> {  
  // 경로의 30% 지점에서 실행됩니다.  
})  
```  
{{< /tab >}}
{{< tab "Kotlin" >}}
```kotlin  
.addDisplacementMarker({ pathLength -> pathLength * 0.3 }) {  
  // 경로의 30% 지점에서 실행됩니다.  
}  
```  
{{< /tab >}}
{{< /tabs >}}

첫 번째 매개변수는 경로의 전체 길이를 제공하는 콜백 함수입니다. 
이를 통해 경로에 대한 명확한 제어를 할 수 있습니다. 
원하는 값을 반환할 수 있으며, 나머지 변위 기반 마커는 이 함수에 기반을 둡니다. 
이 기능을 사용하는 것은 권장되지 않지만, 필요할 경우 사용할 수 있도록 제공됩니다.
