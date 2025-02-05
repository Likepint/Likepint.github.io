---
layout: post
description: > 
  TPS 프로젝트 설정 및 매크로, 
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.02.03 Unreal C++ TPS

![image](https://cdn.htmr.kr/uploads/news/_thumbnail/642e2b3a64553.jpg)

<!--more-->
* toc 
{:toc .large-only}

- ## 필요한 매크로(Macro) 설정

```java
// TPSExample.h

#pragma once

#include "CoreMinimal.h"

DECLARE_LOG_CATEGORY_EXTERN(TPS, Log, All);

#define CALLINFO (FString(__FUNCTION__) + TEXT('(') + FString::FromInt(__LINE__) + TEXT(')'))

#define PRINT_CALLINF() UE_LOG(TPS, Warning, TEXT("%s"), *CALLINFO)

#define PRINT_LOG(fmt, ...) UE_LOG(TPS, Warning, TEXT("%s %s"), *CALLINFO, *FString::Printf(fmt, ##__VA_ARGS__))
```

```java
// TPSExample.cpp

#include "TPSExample.h"
#include "Modules/ModuleManager.h"

IMPLEMENT_PRIMARY_GAME_MODULE( FDefaultGameModuleImpl, TPSExample, "TPSExample" );

DEFINE_LOG_CATEGORY(TPS);
```

```java
// CGameMode_TPS.cpp

#include "CGameMode_TPS.h"
#include "TPSExample.h"

ACGameMode_TPS::ACGameMode_TPS()
{
	PRINT_LOG(TEXT("My Log : %s"), TEXT("TPS Project!!"));

}
```
> 게임모드가 CGameMode_TPS로 설정되어 있다면, 프로젝트 실행 시 로그가 남을 것

- ### 매크로 결과 화면

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfNzEg/MDAxNzM4NTUwNDU0NjQ0.Yfq670ekCK8XyifLwMFdpbMedsCTV6Q42VxPSdPzV8kg.jUXoS1nvNrFPfRcUn5GGvuq4JCaxWxK-Lkos44TxeYIg.PNG/image.png?type=w800)
> __FUNCTION__ ACGameMode_TPS::ACGameMode_TPS(__LINE__), My Log 이름까지 정상적으로 찍힌 모습

- ## TPS 플레이어 초기 설정 및 생성

```java
// ACTPSCharacter.cpp

ACTPSCharacter::ACTPSCharacter()
{
	PrimaryActorTick.bCanEverTick = true;

	// Mesh에 SK_Mannequin 로드 후 설정
	ConstructorHelpers::FObjectFinder<USkeletalMesh> mesh(L"/Script/Engine.SkeletalMesh'/Game/PJS/Characters/Meshes/SK_Mannequin.SK_Mannequin'");
	if (mesh.Succeeded()) // 성공이라면
		GetMesh()->SetSkeletalMesh(mesh.Object);
    
	// 위치값과 회전값 반영
	GetMesh()->SetRelativeLocationAndRotation(FVector(0, 0, -90), FRotator(0, -90, 0));

  	// SpringArm Component 생성
	SpringArm = CreateDefaultSubobject<USpringArmComponent>("SpringArm");
	// Root Component에 Attach
	SpringArm->SetupAttachment(RootComponent);
	// 상대위치 지정
	SpringArm->SetRelativeLocation(FVector(0, 60, 80));
	// TargetArmLength 지정
	SpringArm->TargetArmLength = 300;

	// Camera Component 생성
	Camera = CreateDefaultSubobject<UCameraComponent>("Camera");
	// SpringArm Component에 Attach
	Camera->SetupAttachment(SpringArm);
}
```
- #### 플레이어 생성 결과 화면

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDNfMTU2/MDAxNzM4NTU4NDIyNTUx.LzaNtGMFPVAdIElsTSNJHSvLFBMP90SkMkIdHbrMScog.mlwknUJswAqqtHlIq17Ryfygb9vsW79Di7lXrJnrXAwg.PNG/image.png?type=w800)

- #### SetCharacterBlueprint()

```java
ACTPSCharacter::ACTPSCharacter()
{
	PrimaryActorTick.bCanEverTick = true;

	InitializeCharacter();

}

// ... 생략

void ACTPSCharacter::InitializeCharacter()
{
	// Mesh에 SK_Mannequin 로드 후 설정
	ConstructorHelpers::FObjectFinder<USkeletalMesh> mesh(L"/Script/Engine.SkeletalMesh'/Game/PJS/Characters/Meshes/SK_Mannequin.SK_Mannequin'");
	if (mesh.Succeeded()) // 성공이라면
		GetMesh()->SetSkeletalMesh(mesh.Object);

	// 위치값과 회전값 반영
	GetMesh()->SetRelativeLocationAndRotation(FVector(0, 0, -90), FRotator(0, -90, 0));

	// SpringArm Component 생성
	SpringArm = CreateDefaultSubobject<USpringArmComponent>("SpringArm");
	// Root Component에 Attach
	SpringArm->SetupAttachment(RootComponent);
	// 상대위치 지정
	SpringArm->SetRelativeLocation(FVector(0, 60, 80));
	// TargetArmLength 지정
	SpringArm->TargetArmLength = 300;

	// Camera Component 생성
	Camera = CreateDefaultSubobject<UCameraComponent>("Camera");
	// SpringArm Component에 Attach
	Camera->SetupAttachment(SpringArm);

}
```

## Github Address (TPSExample Repository)
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/SeSAC_TPSExample)
