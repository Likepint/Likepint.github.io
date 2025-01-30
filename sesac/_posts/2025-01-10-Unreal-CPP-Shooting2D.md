---
layout: post
description: > 
  PlayerPawn 제작, 게임모드 및 레벨 설정 \
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.01.10 Unreal C++ Shooting2D
## 플레이어 이동

### Movement 키바인딩
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfNzQg/MDAxNzM4MjAxNzA1MTY4.TVhJocPWm3cVE6aiEwlhffYH4jOO-d-XK_NiI-sF1bYg.STn1Qtesdi387xm2yJFa7455lUkRWbgDJ4JsitNcQS4g.PNG/image.png?type=w800)
> Axis Mappings에 Vertical, Horizontal을 추가

<!--more-->
* toc 
{:toc .large-only}

```c++
UCLASS()
class SHOOTING2D_API ACPlayerPawn : public APawn
{
	GENERATED_BODY()

// ... 생략

private:
	void OnAxisVertical(float InVal);
	void OnAxisHorizontal(float InVal);

private:
	float Speed = 500.0f;
};

```

### 이동 함수

```c++
void ACPlayerPawn::OnAxisVertical(float InVal)
{
	FVector p0 = GetActorLocation();
	FVector direction = GetActorUpVector() * InVal;

	SetActorLocation(p0 + direction.GetSafeNormal() * Speed * GetWorld()->GetDeltaSeconds());
}

void ACPlayerPawn::OnAxisHorizontal(float InVal)
{
	FVector p0 = GetActorLocation();
	FVector direction = GetActorRightVector() * InVal;

	SetActorLocation(p0 + direction.GetSafeNormal() * Speed * GetWorld()->GetDeltaSeconds());
}
```

> 상하, 좌우 이동 함수를 만들고 이동 공식에 맞춰 함수를 정의

```c++
void ACPlayerPawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAxis(L"Vertical", this, &ACPlayerPawn::OnAxisVertical);
	PlayerInputComponent->BindAxis(L"Horizontal", this, &ACPlayerPawn::OnAxisHorizontal);

}
```

> 프로젝트 설정에서 Axis Mappings에 키 바인딩 했기때문에 BindAxis를 사용하여 함수 바인딩

### 이동 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMjk3/MDAxNzM4MjAzNDk5NDMw.8Hunnd4f2GSMMytz_Pmlo_RuwfFgYciToTqnyGHW2TIg.I1Fx_hMHJIZ2E9oUSkuNaQFm-qPpqZ3qqbvRAEA9uRcg.GIF/Movement.gif?type=w966)

---

## 총알(Bullet) 제작

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfODQg/MDAxNzM4MjAzNjczNzEw.r8zOKkkGi_V1NxLDG2w9j0BkVIVWJvk_5P004SchIh4g.eUhBXEvVVmQW3axkywkZumBUZGe5u_oygV4KxN-uhS8g.PNG/image.png?type=w800)
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfNDIg/MDAxNzM4MjAzNjg0Njg2.sFcxOIDX5TtVLb-9vwCQOXGyx6HCzjGdoybM_I1S_RYg.Vkir2IyjkiqgV8SdSv6CNAshgMp3XhpU80roDQ-t7a4g.PNG/image.png?type=w800)
> Actor Class를 상속받아 CBullet Class 생성

```c++
// CBullet.h
UCLASS()
class SHOOTING2D_API ACBullet : public AActor
{
	GENERATED_BODY()

private:
	UPROPERTY(VisibleAnywhere, Category = "Component")
	class UBoxComponent* BoxCollision;

	UPROPERTY(VisibleAnywhere, Category = "Component")
	class UStaticMeshComponent* StaticMesh;

// ... 생략
}
```

```c++
// Cbullet.cpp
ACBullet::ACBullet()
{
	PrimaryActorTick.bCanEverTick = true;

	// BoxComponent 생성
	BoxCollision = CreateDefaultSubobject<UBoxComponent>("BoxCollision");
	this->SetRootComponent(BoxCollision);
	BoxCollision->SetBoxExtent(FVector(50));
	BoxCollision->SetRelativeScale3D(FVector(1, 0.25, 0.25));

	// StaticMeshComponent 생성
	StaticMesh = CreateDefaultSubobject<UStaticMeshComponent>("StaticMesh");
	StaticMesh->SetupAttachment(BoxCollision); // or RootComponent
}
```

> 플레이어폰(PlayerPawn) 생성때와 마찬가지로 박스콜리전과 Cube 생성

### 총알(Bullet) 블루프린트 생성 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTE3/MDAxNzM4MjA0MDkwNzYx.iEkWo43x5q7QfmHym4Aw-aFIorqfvjN6lajDQmcxMNYg.9btNPpvCnXGKlKmD-T-dGdyFnP_smjeVvycysM72Fngg.PNG/image.png?type=w800)

---

### 총알(Bullet) Spawn Position

```c++
UCLASS()
class SHOOTING2D_API ACPlayerPawn : public APawn
{
	GENERATED_BODY()

private:
	UPROPERTY(VisibleAnywhere, Category = "Component")
	class UArrowComponent* FirePosition;

private:
	UPROPERTY(EditAnywhere, Category = "Bullet")
	TSubclassOf<class ACBullet> BulletFactory;

// ... 생략

private:
	void OnFire();
}
```

> FirePosition 추가 후, 해당 컴포넌트 위치에서 Bullet 생성

```c++
ACPlayerPawn::ACPlayerPawn()
{
  // ... 생략

	FirePosition = CreateDefaultSubobject<UArrowComponent>("FirePosition");
	FirePosition->SetupAttachment(RootComponent);
	FirePosition->SetRelativeLocationAndRotation(FVector(0, 0, 100), FRotator(90, 0, 0));
}
```

> 컴포넌트 생성 후, Attach, LocationAndRotation 설정

### Fire 키바인딩

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTI3/MDAxNzM4MjA1ODc1Nzkz.477_lOHHt364_HrCyT0tKixF7xH-_3wYVipMfcO3ehsg.TzYRxjzwR7ykzsasDpcT3RkR-mY3jYmSAKiucmcwSI4g.PNG/image.png?type=w800)

```c++
void ACPlayerPawn::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	PlayerInputComponent->BindAction(L"Fire", IE_Pressed, this, &ACPlayerPawn::OnFire);

}
```

> 프로젝트 설정에서 Action Mappings에 키 바인딩 했기 때문에 BindAction을 이용하여 함수 바인딩

### Bullet 발사 함수

```c++
void ACPlayerPawn::OnFire()
{
	FTransform t = FirePosition->GetComponentTransform();

	GetWorld()->SpawnActor<ACBullet>(BulletFactory, t);
}
```

### Bullet Tick

```c++
void ACBullet::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	FVector P0 = GetActorLocation();
	FVector v = GetActorForwardVector();

	SetActorLocation(P0 + v.GetSafeNormal() * Speed * DeltaTime);

}
```

> 총알은 스폰되는 순간부터 정면방향으로 쭉 날아가도록 설정

### BulletFactory
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfNzgg/MDAxNzM4MjA2ODY0MjIx.T607ZmtKxd_rmQ6vHqn5mS-m_liUehwbmCZDjoMMDQUg.Y7mw6--dGc-k8tH0DuvW38xEH1e1CefxUTpoD0gaot4g.PNG/image.png?type=w800)
> PlayerPawn 블루프린트에서 BulletFactory에 만들어둔 Bullet으로 설정

## 이동, 발사 결과 화면

![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMjgy/MDAxNzM4MjA2NjY1NTk5.aIdPAV6ZzB-EtGYBxgtSj5vhT3bYJWlEFFoidUJivoQg.25DPQSTksseY-h-fmJ0arvEOD1IUAR7M_rmP6RncB7Ig.GIF/mp4_to_gif.gif?type=w966)

## Github
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/CPP_Shooting2D)
