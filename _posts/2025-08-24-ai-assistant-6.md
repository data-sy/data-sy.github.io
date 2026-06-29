---
title: "[AI Assistant #6] 테스트는 통과했는데 실기기에선 안 울린다?! — 가설로 검증한 알람 정책"
date: 2025-08-24 19:56:34 +0900
categories: [타이머 앱 개발기]
tags: [iOS, SwiftUI, 로컬알림, ADR, 회고]
description: "단위 테스트는 통과했지만 실기기에선 안 울리는 알림. 실기기에서 직접 가설을 세워 검증하며 '제목·본문 중 하나는 필수' 같은 정책을 확정하고 알람 기능을 마무리한 주."
---

>### ⏰ 퀵라벨타이머 (QuickLabelTimer)
- **플랫폼·기술**: iOS 앱, SwiftUI
- **앱 컨셉**: 시간을 빠르게 설정할 수 있는 **퀵 타이머**에, ‘왜 맞췄는지’를 기록할 수 있는 **라벨 기능**을 결합한 타이머 앱
- **개발 목적**: AI-Assisted Programming을 실제 서비스 개발에 적용하여 **장단점**을 살펴보고, **서비스 완성 및 운영 경험**을 통해 **설계 능력과 문제 해결 능력을 강화**하고자 함
- **GitHub**: https://github.com/data-sy/quick-label-timer

<br/>
<br/>

## 이번 주에 한 일
<div style="display:flex; gap:8px; align-items:flex-start;">
<div style="flex:1; min-width:0;"><img src="/assets/img/posts/ai-assistant-6/01.png" alt="" style="width:100%; height:auto;" /></div>
<div style="flex:1; min-width:0;"><img src="/assets/img/posts/ai-assistant-6/02.png" alt="" style="width:100%; height:auto;" /></div>
</div>

이번 주에는 드디어 **알람 기능을 마무리**했다! 
지난주에 정리한 정책대로 **'로컬 알림을 짧은 간격으로 연속 울리기'**방식을 적용했고, 그 과정에서 파생된 **UI 정책 수정**까지 함께 진행했다.

<br/>

**[ 주요 작업 요약 ] **

- 로컬 알림 기반 알람 기능 구현
- 소리·진동 UI 정책 수정 (4가지 → 3가지 선택지로 단순화)

<br/>

지난주에 **단위 테스트와 실기기 동작이 달랐던 경험**이 있어서, 이번에는 **실기기에서 직접 가설을 검증**하는 방식을 썼다. 예를 들어, *"제목·본문 없이 sound만 넣은 알림은 배너가 안 뜨고 소리만 울릴 것이다"* 라는 가설을 세웠지만, **실제 결과는 배너 자체가 뜨지 않았다.**

> 
두 케이스는 예약(`pending_count`) 수치는 동일하지만, **배달(`delivered_count`)에서 차이**를 보인다.
>
- **제목, 본문 모두 `nil` 또는 `“”` 일 때:** 예약은 정상적으로 되었지만, 실제 알림은 도착하지 않음 (**`delivered_count=0`**)
![](/assets/img/posts/ai-assistant-6/03.png)
>  
- **본문을 넣었을 때:** 알림이 정상적으로 배달됨 (**`delivered_count=9`**)
![](/assets/img/posts/ai-assistant-6/04.png) 
> 
>


즉, '알림이 정상적으로 울리려면 제목과 본문 중 최소 하나는 반드시 포함되어야 한다'는 사실을 확인할 수 있었다. 코드 단위테스트만 했다면 “예약이 됐으니 성공이다”라고 착각했을 텐데, **실기기 단위 검증 덕분에 미리 문제를 발견할 수 있었다.**
이런 검증 과정을 거쳐 사용 가능한 기능과 불가능한 기능을 정리했고, 그에 따라 **백그라운드, 포그라운드, 엣지케이스 각각에 맞는 정책**을 마련할 수 있었다.

<br/>

또 하나 재미있었던 포인트는 **소리·진동 조합 문제**였다.

원래는 사용자가 소리/진동을 각각 On·Off로 고를 수 있게 하려 했는데, 로컬 알림으로 정책을 바꾸니 **“소리 O·진동 X”**을 선택하면, 무음 모드일 때 **정반대로 “소리 X·진동 O”**이 울리는 상황이 벌어졌다. 😂
혼란스러운 UX이므로 최종적으로는 옵션을 **소리 / 진동 / 무음** 3가지만 제공하는 걸로 단순화했다.

<div style="display:flex; gap:8px; align-items:center;">
<div style="flex:1; min-width:0;"><img src="/assets/img/posts/ai-assistant-6/05.png" alt="" style="width:100%; height:auto;" /></div>
<div style="flex:0 0 auto;"><strong>➔</strong></div>
<div style="flex:1; min-width:0;"><img src="/assets/img/posts/ai-assistant-6/06.png" alt="" style="width:100%; height:auto;" /></div>
</div>


<br/>

<br/>

## 배운 점

- **실기기 테스트의 중요성**
이번 경험은 지난주에 겪었던 **Suspended 상태 이슈**와도 맞닿아 있다. 코드 단위테스트로 예약 성공 여부까지는 확인할 수 있지만, 실제 기기에서는 iOS 정책과 환경 변수 때문에 **코드상으론 정상인데 실제론 동작하지 않는 경우**가 발생했다. 알람 기능 구현 과정을 통해, 핵심 기능은 **실기기 검증을 병행해야 한다**는 걸 확실히 배웠다.
- **문서화의 힘**
ADR 문서로 과정과 근거를 남겨두니, 정책을 바꿀 때 훨씬 수월했다. “왜 이 결정을 했는가”를 설명할 필요 없이 기록을 찾아보고 **맥락을 바로 파악**할 수 있었다. 다음부터도 중요한 변화가 있을 때는 기록을 잘 해두어야겠다.
- **아키텍처의 힘**
지난주에 적용한 클린 아키텍처가 이번 구현에 큰 도움이 됐다. 각 클래스 역할이 명확히 분리돼 있어서, 어떤 기능을 어디에 넣어야 할지 고민이 줄었고, 복잡한 문제도 작은 단위로 나눠 해결할 수 있었다.
    

<br/>

<br/>

## 다음 주에 할 일

- 남은 자잘한 버그들 수정
- 크래시, 접근성 등의 마무리 작업
- **스토어 메타데이터 준비 - 출시 가즈아 🚀**

<br/>

<br/>

### 관련 문서

- [[ADR-012] Suspended 상태 대응을 위한 백그라운드 알람 정책](https://splashy-shear-9ec.notion.site/ADR-012-Suspended-254dbe50bc0380b8b58cc99cb46f83bc?pvs=74)
- [[ADR-013] 백그라운드 알람 정책과의 일관성을 위한 포그라운드 알람 정책](https://splashy-shear-9ec.notion.site/ADR-013-254dbe50bc038094a7a2d7e2cb334b86?pvs=74)


<br/>

