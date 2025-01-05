---
title: "MeepMeep"
description: "자율 주행에 도움이 되는 다른 도구들을 소개합니다."
summary: ""
date: 2023-09-07T16:04:48+02:00
lastmod: 2023-09-07T16:04:48+02:00
draft: false
weight: 701
toc: true
seo:
  title: "" # custom title (optional)
  description: "" # custom description (recommended)
  canonical: "" # custom canonical URL (optional)
  robots: "" # custom robot tags (optional)
---

## 개요

{{< figure
src="images/tools/meepmeep-half-compressed.jpg"
alt="MeepMeep의 스크린샷"
caption="MeepMeep"
>}}

이 글을 작성하는 시점에서 MeepMeep은 Road Runner를 위한 가장 기능이 풍부한 경로 시각화 도구입니다. 
MeepMeep은 객관적으로 가장 많은 기능을 지원하며, 
`TrajectorySequence` API와 마커를 지원하는 유일한 시각화 도구입니다.

LearnRoadRunner에서 사용된 모든 스크린샷과 gif는 MeepMeep으로 생성되었습니다.

### MeepMeep의 기능들:
- **마커 시각화**
- **타임라인 스크러빙**
- **`TrajectorySequence` 지원**
- **커스터마이징 가능한 GUI**
    - 필드 이미지, 색상 테마 등을 변경 가능
    - 개발 중인 드래그 앤 드롭 경로 생성기
    - 다크 모드 색상 테마
- **Android Studio에서 실행 가능**
    - 기존 시즌 코드와 함께 사용
- **직관적인 필드 좌표**

---
# MeepMeep

MeepMeep은 2021년 5월 6일부터 적용된 Road Runner 경로 시퀀스 API에 맞게 설계된 경로 시각화 도구입니다. 경로 시퀀스에 대한 자세한 내용은 [여기](/trajectory-sequence/)에서 확인할 수 있습니다.

LearnRoadRunner에 있는 모든 스크린샷과 GIF는 MeepMeep을 사용하여 생성되었습니다.

---

## 설치 방법

설치 과정은 Android 앱 개발 및 Gradle 모듈에 대한 지식이 필요하여 직관적이지 않을 수 있습니다. 아래 설치 영상을 참고하세요.

<div class="flex justify-center my-8">
   <iframe width="560" height="315" src="https://www.youtube.com/embed/vdn1v404go8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</div>

> **참고:** MeepMeep 2.x.x 이후 API 변경으로 인해 영상 후반부에 설명된 코드는 유효하지 않습니다. 이 API 변경은 다중 봇 지원을 보다 잘 처리할 수 있게 하기 위함입니다.

---

## 사용 방법

위 영상 후반부에서 기본적인 사용법을 다룹니다.  
MeepMeep을 사용하려면 경로 시퀀스의 작동 방식에 대한 이해가 필요합니다. [경로 시퀀스 페이지](/trajectory-sequence/)에서 API를 숙지하시기 바랍니다.

MeepMeep은 별도의 모듈에서 실행되므로, 일반적인 `TeamCode` SDK 코드에서 실행할 수 없습니다. 이는 경로 자체에는 영향을 주지 않으나, 마커 내에서 실행되는 로봇 제어 코드에는 영향을 줄 수 있습니다. 아래와 같이 주석 처리를 권장합니다:

```java
public class MeepMeepTesting {
    public static void main(String[] args) {
        // MeepMeep 인스턴스 선언
        // 필드 크기를 800 픽셀로 설정
        MeepMeep meepMeep = new MeepMeep(800);

        RoadRunnerBotEntity myBot = new DefaultBotBuilder(meepMeep)
                // 필수: 로봇의 제약 설정 (최대 속도, 최대 가속도, 최대 각속도 등)
                .setConstraints(60, 60, Math.toRadians(180), Math.toRadians(180), 15)
                // 선택 사항: 테마 설정. 기본값은 ColorSchemeRedDark()
                .setColorScheme(new ColorSchemeRedDark())
                .followTrajectorySequence(drive ->
                        drive.trajectorySequenceBuilder(new Pose2d(0, 0, 0))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .forward(30)
                                .addDisplacementMarker(() -> {
                                    /* 마커 콜백 안의 모든 내용은 주석 처리하세요 */

                                    // bot.shooter.shoot()
                                    // bot.wobbleArm.lower()
                                })
                                .turn(Math.toRadians(90))
                                .splineTo(new Vector2d(10, 15), 0)
                                .turn(Math.toRadians(90))
                                .build()
                );

        // 필드 이미지 설정
        meepMeep.setBackground(MeepMeep.Background.FIELD_FREIGHTFRENZY_ADI_DARK)
                .setDarkMode(true)
                // 배경 불투명도 (0~1)
                .setBackgroundAlpha(0.95f)
                .addEntity(myBot)
                .start();
    }
}
```

---

## 다중 봇 추가하기

MeepMeep 2.x 버전은 새로운 API와 엔티티 처리 방식을 도입하여 여러 경로를 동시에 실행하고 조율할 수 있습니다. 새로운 `RoadRunnerBotEntity`를 선언하고, `MeepMeep#addEntity(Entity)`를 통해 추가하세요.

```java
public class MeepMeepTesting {
    public static void main(String[] args) {
        MeepMeep meepMeep = new MeepMeep(800);

        // 첫 번째 봇 선언
        RoadRunnerBotEntity myFirstBot = new DefaultBotBuilder(meepMeep)
                // 봇 색상을 파란색으로 설정
                .setColorScheme(new ColorSchemeBlueDark())
                .setConstraints(60, 60, Math.toRadians(180), Math.toRadians(180), 15)
                .followTrajectorySequence(drive ->
                        drive.trajectorySequenceBuilder(new Pose2d(0, 0, 0))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .build()
                );

        // 두 번째 봇 선언
        RoadRunnerBotEntity mySecondBot = new DefaultBotBuilder(meepMeep)
                // 봇 색상을 빨간색으로 설정
                .setColorScheme(new ColorSchemeRedDark())
                .setConstraints(60, 60, Math.toRadians(180), Math.toRadians(180), 15)
                .followTrajectorySequence(drive ->
                        drive.trajectorySequenceBuilder(new Pose2d(30, 30, Math.toRadians(180)))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .forward(30)
                                .turn(Math.toRadians(90))
                                .build()
                );

        meepMeep.setBackground(MeepMeep.Background.FIELD_FREIGHTFRENZY_ADI_DARK)
                .setDarkMode(true)
                .setBackgroundAlpha(0.95f)
                // 선언한 두 봇 추가
                .addEntity(myFirstBot)
                .addEntity(mySecondBot)
                .start();
    }
}
```

---

## 사용 가능한 필드 이미지

_🚧 작업 중 🚧_

[GitHub 폴더](https://github.com/NoahBres/MeepMeep/tree/master/src/main/resources/background)에서 사용할 수 있는 이미지를 확인하고, [Background 클래스](https://github.com/NoahBres/MeepMeep/blob/72034792df9d3221e73923447ccade94bcb38ca8/src/main/kotlin/com/noahbres/meepmeep/MeepMeep.kt#L434)에서 클래스 이름을 확인하세요.

---

## 사용자 정의 색상 테마 만들기

_🚧 작업 중 🚧_

제공된 색상 테마는 [여기](https://github.com/NoahBres/MeepMeep/blob/master/src/main/kotlin/com/noahbres/meepmeep/core/colorscheme/scheme/ColorSchemeRedLight.kt)를 참고하세요.

---

## 사용자 정의 엔티티 추가

_🚧 작업 중: Heno에게 개선된 링 엔티티 샘플 요청 🚧_

또는 [여기](https://gist.github.com/NoahBres/4136d6617e2870b9e76c5d965f923afd)를 참조하세요.