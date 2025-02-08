---
layout: post
description: > 
  Weapon Rifle, Sniper Attach
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.02.06 Unreal C++ TPS
![image](https://cdn.htmr.kr/uploads/news/_thumbnail/642e2b3a64553.jpg)

<!--more-->
* toc 
{:toc .large-only}

- ## 스나이퍼(Sniper)

### 캐릭터 헤더파일에 선언
```java
UCLASS()
class TPSEXAMPLE_API ACTPSCharacter : public ACharacter
{
	GENERATED_BODY()

	// ... 생략

private:
	UPROPERTY(VisibleAnywhere, Category = "GunMesh")
	class UStaticMeshComponent* SniperMesh;
}
```

### 생성 및 초기 설정
```java
void ACTPSCharacter::InitializeCharacter()
{
	// 스나이퍼 스태틱 메시 컴포넌트 등록
	SniperMesh = CreateDefaultSubobject<UStaticMeshComponent>("SniperMesh");

	// 부모 컴포넌트를 Mesh 컴포넌트로 설정
	SniperMesh->SetupAttachment(GetMesh());

	// 스나이퍼 메시 위치 지정
	SniperMesh->SetRelativeLocation(FVector(0, 90, 120));

	// 스나이퍼 메시 스케일 지정
	SniperMesh->SetRelativeScale3D(FVector(0.15));

	ConstructorHelpers::FObjectFinder<UStaticMesh> sniper(L"/Script/Engine.StaticMesh'/Game/PJS/Weapons/Sniper/Mesh/sniper.sniper'");
	if (sniper.Succeeded())
		SniperMesh->SetStaticMesh(sniper.Object);

}
```

### 스나이퍼(Sniper) 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfMTU5/MDAxNzM4ODA2NjY1MTYw.k1315DJ9DQrzUXiupue0F6RwHS0MInufPCoWoHym6Nwg.iak-UbRMAHp-T0js57Q-rER5124l_4EwsKHSTiUzOi0g.PNG/image.png?type=w800)

### 무기 교체
#### IA_Rifle
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfMTcz/MDAxNzM4ODA2OTQ0NjA0.KGvTJa1JYHw6Dk5TvMX6lL3df21eKjdJC6qkJsCum8cg.gmzrwuQKiwy0bovxHRUsXgh2ebF3hbi_7TgD_lnAwYEg.PNG/image.png?type=w800)

#### IA_Sniper
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfMTQ5/MDAxNzM4ODA2OTU3NzUw.Q0m1n-x_sFwvB2LfSKKCo_zgh2Z1cqLgHnySg8TwpzQg.ZMdqwkcwBr0jVrHjlfmeyvyWj1APz8hlYW9oQxAgtX8g.PNG/image.png?type=w800)

#### Mapping Context
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfMzUg/MDAxNzM4ODA2OTI0ODIw.arsiZDzeH9NNBEQhmJGeBe5bv-gaTL4g1X0OkMawrrYg.nwC5kR70TqJW8RzmqauP4PJM3EVpxHZ2flQVSNJKIq8g.PNG/image.png?type=w800)

#### 키 입력

##### InputAction 추가
```java
private: // Input
	UPROPERTY(EditDefaultsOnly, Category = "Input")
	class UInputAction* IA_Rifle;

	UPROPERTY(EditDefaultsOnly, Category = "Input")
	class UInputAction* IA_Sniper;

private:
	// 총의 종류를 구분하는 Bool변수
	bool bRifle = true;
```

##### 각 무기의 교체 함수
```java
void ACTPSCharacter::OnRifle(const FInputActionValue& InVal)
{
	// 라이플이면 TRUE, 스나이퍼면 FALSE
	bRifle = true;

	if (SniperMesh->IsVisible())
		SniperMesh->SetVisibility(false);

	RifleMesh->SetVisibility(true);
}

void ACTPSCharacter::OnSniper(const FInputActionValue& InVal)
{
	// 라이플이면 TRUE, 스나이퍼면 FALSE
	bRifle = false;

	if (RifleMesh->IsVisible())
		RifleMesh->SetVisibility(false);

	SniperMesh->SetVisibility(true);
}
```

##### 키 입력 함수 바인딩
```java
void ACTPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		EnhancedInputComponent->BindAction(IA_Rifle, ETriggerEvent::Started, this, &ACTPSCharacter::OnRifle);
		EnhancedInputComponent->BindAction(IA_Sniper, ETriggerEvent::Started, this, &ACTPSCharacter::OnSniper);

	}
}
```

##### 무기 모두 숨김 처리
```java
void ACTPSCharacter::BeginPlay()
{
	Super::BeginPlay();
	
	// ... 생략

	// 무기 모두 숨김 처리
	RifleMesh->SetVisibility(false);
	SniperMesh->SetVisibility(false);
}
```

##### 무기 교체 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfNDMg/MDAxNzM4ODA4Nzc3MDEw.VvBOPLbdgYxFCR7mq6yp0hei9fES2eOJdxw3yLfmzKYg.uhVGlzbjAbzPKbwlZf8qqBvHgYiUgtrhhXtjg9weclEg.GIF/Change.gif?type=w966)

### 스나이퍼 유저 인터페이스 (SniperUI)
#### 위젯 블루프린트 생성
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfOTcg/MDAxNzM4ODA5MDg3NDIz.XtTg_-QDrn-pj1V5MwsC63-E78LiDMMDAPBYQD8j7EAg.m4J3TZ1tr7E0i8GJkMt5-geBZZWqp0HQVTBbxlQW-g4g.PNG/image.png?type=w800)
> User Widget을 상속받아 WBP_SniperUI 생성

#### 계층 설정
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfMjk1/MDAxNzM4ODA5MzM1Njg1.4-4TJ6AruxeqtQOgt5GMHWF7_6DTXObMQXGhEj9Pg7Ug.mNEHzesXHCit_DAMc9icxkDtqBDZoBaX0QkSsLx6tJsg.PNG/image.png?type=w800)
> Canvas Panel 생성 후 Image를 자식으로 추가

#### Image 추가
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfMjc2/MDAxNzM4ODA5NDY5Mzkw.9Kw3roryYCXxqSoee5IK4pMoXCZHp6ak-A8cX8cpIlIg.0d9EEWEq1hHSvzXYew-huQls3egnAVb9h2l8rlWeiNgg.PNG/image.png?type=w800)
> Image에 사용할 이미지 추가 후 원하는 스타일에 맞게 이미지 조정

#### SniperUI 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDZfMTk2/MDAxNzM4ODA5NDM3MTEx.qjk2lI_mEQ9Gnrb0v9ecu4JnD76l_n27KQrHqA8tD18g.PilLEv9JgkYeQmLfuC12chUtl21kwqve2nv1oVd6tQ4g.PNG/image.png?type=w800)

#### SniperUI 키입력
```java
UCLASS()
class TPSEXAMPLE_API ACTPSCharacter : public ACharacter
{
	GENERATED_BODY()

	// ... 생략

private:
	UPROPERTY(EditDefaultsOnly, Category = "Input")
	class UInputAction* IA_Zoom;

private:
	// 줌 상태인지를 구분하는 Bool변수
	bool bOnZoom = true;
}
```

```java
void ACTPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		// ... 생략
		
		EnhancedInputComponent->BindAction(IA_Zoom, ETriggerEvent::Started, this, &ACTPSCharacter::OnZoom);
		EnhancedInputComponent->BindAction(IA_Zoom, ETriggerEvent::Completed, this, &ACTPSCharacter::OnZoom);

	}
}
```

#### SniperUI 생성 코드
```java
void ACTPSCharacter::BeginPlay()
{
	Super::BeginPlay();
	
	// ... 생략

	// 스나이퍼 UI 위젯 인스턴스 생성
	SniperUI = CreateWidget<UUserWidget>(GetWorld(), SniperUIFactory);
}
```

#### SniperUI 생성 조건 코드
```java
void ACTPSCharacter::OnZoom(const FInputActionValue& InVal)
{
	// 스나이퍼건 모드가 아니라면 처리X
	if (bRifle) return;
	
	if (bOnZoom == false)
	{// Started 입력처리
		bOnZoom = true;

		SniperUI->AddToViewport();

		// 카메라의 시야각 FOV 설정
		Camera->SetFieldOfView(45);
	}
	else
	{// Completed 입력처리
		bOnZoom = false;

		SniperUI->RemoveFromParent();

		Camera->SetFieldOfView(90);
	}
}
```
> 현재 총이 Sniper인지 판단 후 Zoom(Mouse Right Button) 클릭 시 OnZoom() 호출

#### SniperUI 생성 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfNzkg/MDAxNzM4ODkwMTEyNTgz.dcdA5MEJ2YGk9M8oNe1IyQpuTimcOFpdZFRT4dwWz64g.Xq-vkaabCFsebk9LS33ZKTNt7187Yk-fyojRsna_9kEg.GIF/SniperZoom.gif?type=w966)

#### 발사(Fire)

##### OnFire()
```java
void ACTPSCharacter::OnFire(const FInputActionValue& InVal)
{
	// Rifle
	if (bRifle)
	{
		FTransform firePosition = RifleMesh->GetSocketTransform ( "FirePosition" );

		GetWorld ( )->SpawnActor<ACBullet> ( BulletFactory , firePosition );
	}
	// Sniper
	else
	{
		// LineTrace의 시작 위치
		FVector start = Camera->GetComponentLocation();
		 
		// LineTrace의 종료 위치
		FVector end = start + Camera->GetForwardVector() * 5000;
		
		// LineTrace의 충돌 정보를 담을 변수
		FHitResult info;

		// 충돌 옵션 설정 변수
		FCollisionQueryParams params;

		// 플레이어 제외 설정
		params.AddIgnoredActor(this);

		// Channel 필터를 이용한 LineTrace 충돌 검출
		bool bHit = GetWorld()->LineTraceSingleByChannel(info, start, end, ECollisionChannel::ECC_Visibility, params);

		// LineTrace가 충돌했을 때
		if (bHit)
		{
			// 충돌 처리 -> 충돌효과 표현
			
			// 총알 파편 효과 출력할 트랜스폼
			FTransform t;

			// 부딪힌 위치 할당
			t.SetLocation(info.ImpactPoint);

			// 총알 파편 효과 인스턴스 생성
			UGameplayStatics::SpawnEmitterAtLocation(GetWorld(), BulletEffectFactory, t);

			// 충돌한 오브젝트의 컴포넌트
			auto hitComp = info.GetComponent();

			// 해당 컴포넌에 물리가 적용되어 있다면
			if (hitComp and hitComp->IsSimulatingPhysics())
			{
				// 조준한 방향
				FVector direction = (end - start).GetSafeNormal();

				// 날려버릴 힘 ( F = ma )
				FVector force = direction * hitComp->GetMass() * 500000;

				// 그 방향으로 날리기
				hitComp->AddForceAtLocation(force, info.ImpactPoint);
			}
		}
	}
}
```
> 라이플인지 아닌지를 판단하여 스나이퍼인 경우 LineTrace를 사용하여 발사 \
> 그리고 충돌한 오브젝트가 물리가 적용되어 있을 경우 충격에 반응하여 날아가도록 설정

##### 스나이퍼(Sniper) 총알 효과
```java
private:
	UPROPERTY ( EditAnywhere , Category = "BulletEffect" )
	class UParticleSystem* BulletEffectFactory;
```

##### 스나이퍼(Sniper) 발사 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMTA0/MDAxNzM4ODkzODk0MzEx.w9zq14SVzeGA4dpx_U2SPeLTYdaelr7hY6g8Ba_HXTcg.Pk1We31kPpW7Gx53PqWRODK-98jJHPUsP3dqGBhhZxog.GIF/SniperFire.gif?type=w966)

### CrossHairUI
#### CrssHairUI 생성 코드
```java
UCLASS()
class TPSEXAMPLE_API ACTPSCharacter : public ACharacter
{
	GENERATED_BODY()

	// ... 생략

private:
	// 일반 조준 크로스헤어UI 위젯 팩토리
	UPROPERTY(EditDefaultsOnly, Category = "SniperUI")
	TSubclassOf<class UUserWidget> CrosshairUIFactory;

	// 일반 조준 크로스헤어UI 위젯 인스턴스
	UPROPERTY()
	class UUserWidget* CrossHairUI;
}
```
```java
void ACTPSCharacter::BeginPlay()
{
	Super::BeginPlay();
	
	// ... 생략

	// 일반 조준 크로스헤어UI 위젯 인스턴스 생성
	CrossHairUI = CreateWidget<UUserWidget>(GetWorld(), CrosshairUIFactory);

	// 일반 조준 UI 등록
	CrossHairUI->AddToViewport();
}
```

#### CrossHairUI 생성 조건 코드
```java
void ACTPSCharacter::OnZoom(const FInputActionValue& InVal)
{
	// 스나이퍼건 모드가 아니라면 처리X
	if (bRifle) return;
	
	if (bOnZoom == false)
	{// Started 입력처리
		bOnZoom = true;

		SniperUI->AddToViewport();

		// 카메라의 시야각 FOV 설정
		Camera->SetFieldOfView(45);

		CrossHairUI->RemoveFromParent();
	}
	else
	{// Completed 입력처리
		bOnZoom = false;

		SniperUI->RemoveFromParent();

		Camera->SetFieldOfView(90);

		CrossHairUI->AddToViewport();
	}

}
```
> 스나이퍼 줌을 키면 크로스헤어는 지워주고, 줌을 끄면 다시 생성하도록 설정

#### CrossHairUI 생성 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfNjUg/MDAxNzM4ODk0NzU3Nzg0.Ku9b7BlqRO5qTyzF1rpFTDf4tbrM5XdurJwVJmvUjPog.htxCqUP8QAtXi6Sl1nCx6Sg6IEh2uunQgFKWMUvmmPUg.GIF/CrossHairUI.gif?type=w966)


## Github Address (TPSExample Repository)
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/SeSAC_TPSExample)
