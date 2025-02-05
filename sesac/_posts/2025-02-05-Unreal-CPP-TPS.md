---
layout: post
description: > 
  Weapon Rifle, Sniper Attach
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.02.05 Unreal C++ TPS
![image](https://cdn.htmr.kr/uploads/news/_thumbnail/642e2b3a64553.jpg)

<!--more-->
* toc 
{:toc .large-only}

- ## 총알(Bullet)
### CBullet Class 생성
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfNjQg/MDAxNzM4NzE5MzQzOTY1.A3Sh2KJA42c2KSHMX7is60a4TRffjP8PwkNwXw5vkssg.SvV3fTDdH3UQ6AzMCEBvKxmyixwxYdx14b88gPpJa9Yg.PNG/image.png?type=w800)
> Actor Class로부터 상속

### 총알의 Collision과 StaticMesh 생성
#### Add in CBullet.h
```java
UCLASS()
class TPSEXAMPLE_API ACBullet : public AActor
{
	GENERATED_BODY()

private:
	// 충돌 컴포넌트
	UPROPERTY(VisibleAnywhere, Category = "Component")
	class USphereComponent* SphereCollision;

	// 외관 컴포넌트
	UPROPERTY(VisibleAnywhere, Category = "Component")
	class UStaticMeshComponent* StaticMesh;
}
```

#### Add in CBullet.cpp
```java
ACBullet::ACBullet()
{
	PrimaryActorTick.bCanEverTick = true;

	// 충돌체 등록
	SphereCollision = CreateDefaultSubobject<USphereComponent>("SphereCollision");

	// 충돌 프로파일 설정
	SphereCollision->SetCollisionProfileName("BlockAll");

	// 충돌체 크기 설정
	SphereCollision->SetSphereRadius(20);

	// 루트컴포넌트로 설정
	SetRootComponent(SphereCollision);

	// Static(외관) 컴포넌트 등록
	StaticMesh = CreateDefaultSubobject<UStaticMeshComponent>("StaticMesh");

	// 부모 컴포넌트 지정
	StaticMesh->SetupAttachment(RootComponent);

	// 충돌 비활성화
	StaticMesh->SetCollisionEnabled(ECollisionEnabled::NoCollision);

	// 외관 크기 설정
	StaticMesh->SetRelativeScale3D(FVector(0.35));

	// 외관 위치 설정
	StaticMesh->SetRelativeLocation(FVector(0, 0, -17));

}
```

#### CBullet 블루프린트 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfMTEw/MDAxNzM4NzE5OTk0MDY0.Pg3i-6nqWpMO5j82U_39c1lrEwsx1frXY5FwSAjHp6og.cW0-LudHiRcuSwZjNnTVBFY5V84PkclRxKUZIe_j2WUg.PNG/image.png?type=w800)

### 발사체 컴포넌트 (Projectile Component)

#### Add in CBullet.h
```java
private:
	// 발사체의 이동을 담당할 컴포넌트
	UPROPERTY(VisibleAnywhere, Category = "Movement")
	class UProjectileMovementComponent* Projectile;
```

#### Add in CBullet.cpp
```java
ACBullet::ACBullet()
{
	PrimaryActorTick.bCanEverTick = true;

	// ... 생략

	// 발사체 컴포넌트 등록
	Projectile = CreateDefaultSubobject<UProjectileMovementComponent>("Porjectile");

	// 컴포넌트가 갱신시킬 컴포넌트 지정
	Projectile->SetUpdatedComponent(SphereCollision);

}
```

#### 컴포넌트 기초 설정
```java
ACBullet::ACBullet()
{
	PrimaryActorTick.bCanEverTick = true;

	// ... 생략

	// 초기 속도
	Projectile->InitialSpeed = 5000;

	// 최대 속도
	Projectile->MaxSpeed = 5000;

	// 반동 여부
	Projectile->bShouldBounce = true;

	// 반동 값
	Projectile->Bounciness = 0.3;

}
```

#### 발사체 컴포넌트 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfNjEg/MDAxNzM4NzIwMjAxNTcx.4GlJJjjOPXid4FZcQmZdjXVWkmY7Fo_G1GdK26jme00g.G9MKCi1-zD-WurwOCWlXFNbEKbrsBfyXDghO26ynlYMg.PNG/image.png?type=w800)

### 총알 테스트 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfMjQ2/MDAxNzM4NzIzNTkxNzAz.gSemYmKVVUDLSzOvm_Q-fY44FgV27ZzqeT_EFo04HxUg.E78Jc-JbHmDGaboSbvJOy6MyS-tGE6iLK8lxRruxDzYg.GIF/Bullet.gif?type=w966)
> 발사체처럼 날아가고, 바운드도 되는 모습

## 라이플(Rife)
### Add in TPSCharacter.h
```java
UCLASS()
class TPSEXAMPLE_API ACTPSCharacter : public ACharacter
{
	GENERATED_BODY()

private:
	UPROPERTY(VisibleAnywhere, Category = "RifleMesh")
	class USkeletalMeshComponent* RifleMesh;
}
```

### Add in TPSCharacter.cpp
```java
void ACTPSCharacter::InitializeCharacter()
{
	// ... 생략

	// 부모 컴포넌트를 Mesh 컴포넌트로 설정
	RifleMesh->SetupAttachment(GetMesh());

	// 스켈레탈 메시 데이터 로드
	ConstructorHelpers::FObjectFinder<USkeletalMesh> rifle(L"/Script/Engine.SkeletalMesh'/Game/PJS/Weapons/Rifle/Mesh/SK_FPGun.SK_FPGun'");
	if (rifle.Succeeded())
		RifleMesh->SetSkeletalMesh(rifle.Object);

}
```

### 라이플(Rifle) 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfMzUg/MDAxNzM4NzIzNTExMDA1.eutWJAVAw5FFNOvkXx4FGCQ7DhJkA3phCcNwYFGazkAg.oLQt9cuGHPsLnn5UBlqaHGfF6eeDgcm-BRZShENDjPQg.PNG/image.png?type=w800)

### 발사(Fire)
#### IA_Fire 생성, 설정
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfMjE1/MDAxNzM4NzIyMTM1MzA0.1ehnSucBYyvjsJqG25WbaXm-rP0n3zCHvlQ4QirJ2nsg.fTD46bQvPATquNqq3xNDzfoyU8DS3vDuQLGrL9gqxrcg.PNG/image.png?type=w800)

#### Mapping Context 설정
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfMjk0/MDAxNzM4NzIyMjY2MzU4.LVKGmoA6jUEGWSoGPoFv1FZJqYu5lp6k2D6Rfi1e-KYg.KJiZMgUuwpBcoKL02veE8ji_XpzUTlqVCrC499rQnYsg.PNG/image.png?type=w800)

#### 캐릭터 입력 바인딩
```java
void ACTPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		EnhancedInputComponent->BindAction(IA_Fire, ETriggerEvent::Started, this, &ACTPSCharacter::OnFire);

	}
}
```

#### OnFire()
```java
void ACTPSCharacter::OnFire(const FInputActionValue& InVal)
{
	FTransform firePosition = RifleMesh->GetSocketTransform("FirePosition");

	GetWorld()->SpawnActor<ACBullet>(BulletFactory, firePosition);

}
```

#### 생명 주기 설정
```java
ACBullet::ACBullet()
{
	PrimaryActorTick.bCanEverTick = true;

	// ... 생략

	// 생명 시간 주기
	InitialLifeSpan = 2;

}
```

##### 타이머(함수)를 사용한 생명 주기 설정
```java
void ACBullet::BeginPlay()
{
	Super::BeginPlay();

	FTimerHandle deathTimer;
	//                                     타이머핸들,  객체, 바인딩할 함수, 주기, 반복여부
	GetWorld()->GetTimerManager().SetTimer(deathTimer, this, &ACBullet::Die, 2, false);

}

void ACBullet::Die()
{
	Destroy();

}
```

#### 타이머(람다)를 사용한 생명 주기 설정
```java
void ACBullet::BeginPlay()
{
	Super::BeginPlay();
	
	FTimerHandle deathTimer;
	GetWorld()->GetTimerManager().SetTimer
	(deathTimer,
		FTimerDelegate::CreateLambda // 람다
		(
			[this]()->void // [] 안에 구현부에서 사용할 객체 또는 변수, 리턴 타입
			{
				Destroy ( ); // 함수의 구현부
			}
		),
	2, false);

}
```

##### 람다(Lambda)란?
![image](https://modoocode.com/img/1152175050EB03B514EB55.webp)

** [모두의 코드 - 람다(lambda) 함수](https://modoocode.com/196) **


#### Fire() 구현 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDVfMTIz/MDAxNzM4NzMwODE2OTUw.o7VGDW_miY_Jt77CcgX98WibScSj7pg-0umXgruiZe8g.Nf_KShLCGMzLNiMgME1egvJ-Afz5dlHh24mClrOLgqog.GIF/Fire.gif?type=w966)

## Github Address (TPSExample Repository)
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/SeSAC_TPSExample)
