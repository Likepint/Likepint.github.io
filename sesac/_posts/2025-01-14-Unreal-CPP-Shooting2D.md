---
layout: post
description: > 
  충돌 처리 (Collision) 적용
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.01.14 Unreal C++ Shooting2D

![image](https://cdn.htmr.kr/uploads/news/_thumbnail/642e2b3a64553.jpg)

<!--more-->
* toc 
{:toc .large-only}

## 충돌 처리 (Collision)

- ### 1. 오브젝트 채널 (Object Channels) 생성
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfMTUg/MDAxNzM4NTQyMjk0ODE0.NKlav9ur-8CwoD2yXB8dFxjA50Uzz5id-bleI5-VHncg.UWEmX6BlPQLZcDHvNhrNXmQNNUw_dRpAkP6pOnF4PjYg.PNG/image.png?type=w800)
> **오브젝트명과 기본응답방법을 설정 (Default Response는 Ignore로 설정)**

<!--more-->
* toc 
{:toc .large-only}

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfMjUg/MDAxNzM4NTQzNjIwNDQ0.IihGc2z9sF1qQ-zTam1LviK64wIE3aeGhCEsSB-Z4gog.7fiN-n83LeHdvqb-CDrsFYWvlrqMmd9OFkAGlssOtJkg.PNG/Collision.png?type=w800)
> 오브젝트 타입을 추가해주면 Config폴더에 DefaultEngine.ini 파일이 변경되면서 코드로 추가됨

- ### 2. 오버랩 이벤트 활성화 및 채널응답 설정
```c++
ACPlayerPawn::ACPlayerPawn()
{
	PrimaryActorTick.bCanEverTick = true;

	// ... 생략

	// 오버랩 이벤트 활성화
	BoxCollision->SetGenerateOverlapEvents(true);

	// 충돌 응답 QueryAndPhysics로 설정
	BoxCollision->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);

	// Object Type 1번(Player)으로 설정
	BoxCollision->SetCollisionObjectType(ECollisionChannel::ECC_GameTraceChannel1);

	// 모든 채널 무시
	BoxCollision->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Ignore);

	// Enemy 채널만 오버랩 활성화
	BoxCollision->SetCollisionResponseToChannel(ECollisionChannel::ECC_EngineTraceChannel2, ECollisionResponse::ECR_Overlap);
}
```

- #### Collision 설정 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfMTYy/MDAxNzM4NTQ0MjE1Mjg5.kS8_45mUyBBH_m4hBRRDbx3OJ0GEHGEersuyfrvYWqUg.ws8kBg7FrciHEZF39tjNjbaKVnGTVT0IiIdROlGAVfYg.PNG/image.png?type=w800)
> 코드대로 설정이 된다면 성공 (적용이 되지 않는다면 부모클래스를 바꿨다가 바꿔주도록)

- ### etc. 프리셋으로 설정
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfMzkg/MDAxNzM4NTQ0NDE5ODc1.5rErtxs-t7B_bij4Z7uVq_exaFV0nZqFmGqL93HriqUg.E6WjppFSFK9kyILMW9QLclzX6yTjk4yqLwQZfN28_NMg.PNG/image.png?type=w800)
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfMzAg/MDAxNzM4NTQzMDk3NjY2.rY6gbjv4fP2R7cml587KE0MrKtKG4ctTRE3vqS-UORgg.HuR8h9_cof3pSpB2uN88qtSCCCOhCMzx58Vm8Toe1-Qg.PNG/CollisionPreset.png?type=w800)
```c++
ACPlayerPawn::ACPlayerPawn()
{
	PrimaryActorTick.bCanEverTick = true;

	// ... 생략

	// 오버랩 이벤트 활성화
	BoxCollision->SetGenerateOverlapEvents(true);

	// 프리셋으로 설정
	BoxCollision->SetCollisionProfileName("Player");

	//// 충돌 응답 QueryAndPhysics로 설정
	//BoxCollision->SetCollisionEnabled(ECollisionEnabled::QueryAndPhysics);

	//// Object Type 1번(Player)으로 설정
	//BoxCollision->SetCollisionObjectType(ECollisionChannel::ECC_GameTraceChannel1);

	//// 모든 채널 무시
	//BoxCollision->SetCollisionResponseToAllChannels(ECollisionResponse::ECR_Ignore);

	//// Enemy 채널만 오버랩 활성화
	//BoxCollision->SetCollisionResponseToChannel(ECollisionChannel::ECC_EngineTraceChannel2, ECollisionResponse::ECR_Overlap);
}
```
> 이전 코드를 주석으로 처리하고 프리셋으로 설정

- #### Collision Presets 설정 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfNzIg/MDAxNzM4NTQ0NTM2ODk2.bvaJS906llAA3kH_3qJ28ZwpSIPj_7fihnUD_WLUtmUg.vsK8-DlwnXKXevGXcbRDzb2tElCsJj9nXDDTfs30k48g.PNG/image.png?type=w800)

> Collision Presets가 코드대로 적용된다면 프리셋 설정대로 설정됨

## Github
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/CPP_Shooting2D)
