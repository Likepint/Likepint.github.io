---
layout: post
description: > 
  PlayerPawn 제작, 게임모드 및 레벨 설정
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.01.09 Unreal C++ Shooting2D
## 플레이어(PlayerPawn) 제작

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMjlfNDQg/MDAxNzM4MTU3ODk5MDMw._cVxRSGlF_IINinkjoKC28AAilC6LsrX1zka6mdrirQg.1d-R6pk-QA2GQh0v1l-XFkiGaLUF5QTlSvvyRDwc8_4g.PNG/image.png?type=w800)
> **Pawn class**를 상속받아 플레이어 폰 생성 (초기에는 Cube를 사용하여 제작 예정)

<!--more-->
* toc 
{:toc .large-only}

```c++
private:
  UPROPERTY(VisibleAnywhere, Category = "Component")
  class UBoxComponent* BoxCollision;

  UPROPERTY(VisibleAnywhere, Category = "Component")
  class UStaticMeshComponent* StaticMesh;
  
```
> UBoxComponent를 사용하여 BoxCollision 생성 (충돌 판단)\
> UStaticMeshComponent를 사용하여 StaticMesh 생성

**`⁉️전방 선언과 포인터로 사용하는 이유!!`** \
`전방 선언을 사용하여 해당 클레스의 존재만 인식 시켜주기 위한 방법` \
`헤더 파일의 변경이 일어난다면, Include된 모든 헤더 파일들이 Re-Compile 발생` **`(의존성 관련)`** \
**`하지만,`** `포인터로 선언하지 않는다면 객체의 크기를 정확하게 파악할 수 없어 할당 불가능` \
`포인터의 경우, 운영체제 32비트의 경우 4 byte, 64비트의 경우 8 byte 사용`

```c++
#include "CPlayerPawn.h"
#include "Components/BoxComponent.h"
#include "Components/StaticMeshComponent.h"

ACPlayerPawn::ACPlayerPawn()
{
	PrimaryActorTick.bCanEverTick = true;

	// BoxComponent 생성
	BoxCollision = CreateDefaultSubobject<UBoxComponent>("BoxCollision");
	this->SetRootComponent(BoxCollision);
	BoxCollision->SetBoxExtent(FVector(50));

	// StaticMeshComponent 생성
	StaticMesh = CreateDefaultSubobject<UStaticMeshComponent>("StaticMesh");
	StaticMesh->SetupAttachment(BoxCollision); // or RootComponent

}
```
> Component는 CreateDefaultSubobject<Type>(FName, bool) 함수를 사용하여 생성

```c++
template<class TReturnType>
TReturnType* CreateDefaultSubobject(FName SubobjectName, bool bTransient = false)
{
	UClass* ReturnType = TReturnType::StaticClass();
	return static_cast<TReturnType*>(CreateDefaultSubobject(SubobjectName,
                                                          ReturnType,
                                                          ReturnType,
                                                          /*bIsRequired =*/ true,
                                                          bTransient)
  );

}
```
### 플레이어 제작 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMjlfMjUg/MDAxNzM4MTU5MTExOTY2.DKZVvqbv02niVt1Aq4-OceukK02TJsBIvw5puisGOWgg.2NCug-UHaEgdb-kdY-gyBSPo56Y6czv0X68le0PE3kYg.PNG/image.png?type=w800)

---

## 게임모드 설정
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMjlfMjY2/MDAxNzM4MTU5MzkxNjEw.wxLGNCMrAUhO6U7vCULHYtZgXL_aO8oPNni0UFCoYg8g.pMvs3zg_bh3IxP8XgIefyf1xssQb9qjbKJKyKMxG0akg.PNG/image.png?type=w800)
> GameModeBase Class를 상속받아 새로운 게임모드 재정의

```c++
#include "CGameMode.h"

ACGameMode::ACGameMode()
{
  ConstructorHelpers::FClassFinder<APawn>
    pawn(L"/Script/Engine.Blueprint'/Game/BP_CPlayerPawn.BP_CPlayerPawn_C'");
  if (pawn.Succeeded())
  	DefaultPawnClass = pawn.Class;

}
```
> **ConstructorHelpers::FClassFinder** 사용하여 **DefaultPawnClass**로 지정할 플레이어 블루프린트 로드 \
> 해당 클래스를 **성공적으로 불러왔다면**, **DefaultPawnClass**로 지정

### DefaultPawnClass 설정 결과
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMjlfNDUg/MDAxNzM4MTU5NDE0MTEz.JIN6gNrtAwQOdIhwR0XWxqUYd3Tlu2hluFKjlPJxyw8g.-XylN4q6FdFuGduu6Fl8f78iijpG80dgwwgKTDIDITEg.PNG/image.png?type=w800)

---

## 레벨 설정
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMjlfMjc1/MDAxNzM4MTU5NDI2NTk1.Qhxx8jkW_CgGU51HOjQ2H579nyCSP8ntXJizlHXB7ZAg.wCDuHyhmH4CTKI-orGXTnRQHIAgSdPo6jwHh3ZcNr5Eg.PNG/image.png?type=w800)
> PlayerStart : FVector(0, 0, 0) \
> Directional Light : FVector(-1500, 0, 0) \
> CameraActor : FVector(-1000, 0, 0), Orthographic 설정, OrthoWidth 1920, AutoActivateforPlayer Player0

---

## 이동 테스트 코드 (오른쪽으로 계속 이동)
```c++
void ACPlayerPawn::Tick(float DeltaTime)
{
  Super::Tick(DeltaTime);

  // 플레이어가 계속 오른쪽 이동
  // 이동 공식 : P = P0 + v(direction * speed) * t
  FVector P0 = GetActorLocation();
  FVector velocity = GetActorRightVector() * 500.0f;

  SetActorLocation(P0 + velocity * DeltaTime);

}
```

### 이동 테스트 코드 결과
![gif)](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMjlfMjUg/MDAxNzM4MTYwNDc1MDY5.WpiTriZNFcbXzyp-tUWB8GDu4zZOtDf8VKHH6iHhW90g.bNG3eVCjT_LVgy6Bx7CGYCptVXkTkSHXwoW2mHhLyH0g.GIF/2025-01-29_10-01-26.gif?type=w966)

## Github
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/CPP_Shooting2D)
