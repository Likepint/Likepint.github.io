---
layout: post
description: > 
  Enemy 생성, 확률적으로 플레이어 방향으로 향하도록 설정, EnemyFactory, Timer 사용
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.01.13 Unreal C++ Shooting2D
## Enemy Class

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTg1/MDAxNzM4MjI3ODMyNTEz.1u_zaIRe08ud8mfzxfSnnGydSE5YWTy5fPO_ItQptNgg.QqDOO5tBCsMFrzgcfiA7wstYZSoq4dxOWwVfuOYiB5Yg.PNG/image.png?type=w800)

<!--more-->
* toc 
{:toc .large-only}

![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTM3/MDAxNzM4MjI3ODQ4MTcz.oROAXlTJ2ZFSIgrbqi9lFXYn-7GoB3y2Sd22qeTUSDYg.FS0RV585ojWNdwXQnjv-lx5TLI7f0Xmqhy5R3QmqWagg.PNG/image.png?type=w800)
> Actor Class를 상속받아 CEnemy Class 생성

### Enemy 코드
```c++
private:
	UPROPERTY(VisibleAnywhere, Category = "Component")
	class UBoxComponent* BoxCollision;

	UPROPERTY(VisibleAnywhere, Category = "Component")
	class UStaticMeshComponent* StaticMesh;
```
> Enemy도 PlayerPawn과 동일하게 BoxCollision과 StaticMesh로 생성

```c++
ACEnemy::ACEnemy()
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
> PlayerPawn 생성과 동일

### Enemy 블루프린트 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzFfMjcx/MDAxNzM4Mjg2NzQ1Nzk0.B1F_bdOcBr3FWHQxagcEY4zQjN98SvT4GBEyfABdbt8g.WRKmqF64MAYlkJj93C6Qscdv3LS7DvAS3gvTJs6zJkkg.PNG/image.png?type=w800)
> 플레이어와 대비되도록 빨간색 머터리얼을 사용하여 설정

### 테스트코드 (스폰 후 위에서 아래로 계속해서 이동)
```c++
void ACEnemy::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	// 스폰 후 계속해서 밑으로 이동
	// 이동 공식 : P = P0 + v(direction * speed) * t
	FVector P0 = GetActorLocation();
	FVector direction = GetActorUpVector() * -1;

	SetActorLocation(P0 + direction.GetSafeNormal() * Speed * DeltaTime);

}
```
### 테스트코드 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfNjQg/MDAxNzM4MjI5OTQ1NDYy.SppiyupLJGcxmU0wTfNph42kr5GCEWwVvQUYYdwX_Gog.9zC1xRffeT-RGe2a9TAplsNQcuYU0lN-2GgJp_RBO3wg.GIF/d.gif?type=w966)

### 스폰 시 30%의 확률로 플레이어를 향해 이동하도록 설정
#### 필요한 변수들
```c++
private:
	FVector Direction;

	int32 RateTo = 30;
```

#### 30% 확률로 플레이어를 향해 이동하도록 설정
```c++
void ACEnemy::BeginPlay()
{
	Super::BeginPlay();

	// 랜덤한 정수값
	if (FMath::RandRange(1, 100) <= 30)
	{
		// World에 있는 첫번째 컨트롤러에 빙의된 Pawn을 로드
		if (APawn* Target = UGameplayStatics::GetPlayerPawn(GetWorld(), 0))
			// 플레이어를 향하는 방향 벡터
			Direction = (Target->GetActorLocation() - this->GetActorLocation()).GetSafeNormal();
	}
		// 위에서 아래로 향하는 방향 벡터
	else Direction = GetActorUpVector() * -1;

}
```

```c++
void ACEnemy::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	// 이동 공식 : P = P0 + v(direction * speed) * t
	FVector P0 = GetActorLocation();
	SetActorLocation(P0 + Direction * Speed * DeltaTime);

}
```

#### 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzFfMTAg/MDAxNzM4Mjg2MTY2MTQw.qCpA44hUs54EmJEzSx_msFeXTFXM_LXjb_ysN5_FltQg.90Pl0qRnTm2U2ECWvKPBuEZRwRkM9TMouAwA59X8cswg.GIF/d.gif?type=w966)

## EnemyFactory (계속해서 Enemy를 Spawn해주는 Factory)

### EnemyFactory Class 생성
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTg1/MDAxNzM4MjI3ODMyNTEz.1u_zaIRe08ud8mfzxfSnnGydSE5YWTy5fPO_ItQptNgg.QqDOO5tBCsMFrzgcfiA7wstYZSoq4dxOWwVfuOYiB5Yg.PNG/image.png?type=w800)
![alt text](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzFfMjM2/MDAxNzM4Mjg3NDY4ODE5.r1j49Vn0RCbHeBOxZndtZ4MuSfVUL1K1oMaSFEcVSmgg.GmJX18pqNWlcIB10Xr2EZzJg_sNf-Kt40XyMYZXvhe8g.PNG/image.png?type=w800)
> Actor Class를 상속받아 EnemyFactory Class 생성

#### CEnemyFactory.h 코드
```c++
UCLASS()
class SHOOTING2D_API ACEnemyFactory : public AActor
{
	GENERATED_BODY()

private:
	UPROPERTY(VisibleAnywhere, Category = "Component")
	class USceneComponent* SceneComponent;

private:
	UPROPERTY(EditAnywhere, Category = "EnemyFactory")
	TSubclassOf<class ACEnemy> EnemyFactory;

// ... 생략	

private:
	float CurrentTime = 0.0f;
	float SpawnTime = 0.3f;

};
```

#### CEnemyFactory.cpp 코드
```c++
ACEnemyFactory::ACEnemyFactory()
{
	PrimaryActorTick.bCanEverTick = true;

	SceneComponent = CreateDefaultSubobject<USceneComponent>("SceneComponent");

}

// ... 생략

void ACEnemyFactory::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	CurrentTime += DeltaTime;

	if (CurrentTime >= SpawnTime)
	{
		ACEnemy* enemy = GetWorld()->SpawnActor<ACEnemy>(EnemyFactory, SceneComponent->GetComponentTransform());

		CurrentTime = 0.0f;
	}
}
```
> 생성할 Enemy Class를 블루프린트에서 넣어주면 그 Enemy를 SpawnTime이 되면 계속 해서 생성

#### 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzFfODcg/MDAxNzM4Mjg4MzY1MjEy.YCG8iQv7Qv847Daqn_id2-Y_zAR66PDRRnf3ZCMee70g.iYX3KwJ9q0b_CYgaGpFEzowYY5KMEpEZoevXhpis7uYg.GIF/mp4_to_gif.gif?type=w966)

### EnemyFactory 하나만을 사용하여 좌우의 랜덤한 위치에 스폰
#### CEnemyFactory.cpp Tick 코드
```c++
void ACEnemyFactory::Tick(float DeltaTime)
{
	Super::Tick(DeltaTime);

	CurrentTime += DeltaTime;

	if (CurrentTime >= SpawnTime)
	{
		FTransform transform = SceneComponent->GetComponentTransform();

		float y = FMath::FRandRange(-500.0f, 500.0f);

		transform.SetLocation(FVector(transform.GetLocation().X, y, transform.GetLocation().Z));

		ACEnemy* enemy = GetWorld()->SpawnActor<ACEnemy>(EnemyFactory, transform);

		CurrentTime = 0.0f;
	}
}
```
> 랜덤한 좌우(y) 값을 생성하여 Location 지정

#### 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzFfMjkg/MDAxNzM4Mjg5MDQ4MDA4.8oMsLdiDXGmlfwzbqosgJ0-u3brOhAT29K5CzVFoSc4g.BWUQxcVW8j_iyW487o8UVp1-JMs0yi_1pSUnl-o5hoIg.GIF/mp4_to_gif_(1).gif?type=w966)

### Timer를 사용하여 스폰
```c++
// ACEnemyFactory.h
private:
	FTimerHandle TimerHandle;

// ACEnemyFactory.cpp BeginPlay()
void ACEnemyFactory::BeginPlay()
{
	Super::BeginPlay();
	
	GetWorld()->GetTimerManager().SetTimer
	(
		// 바인딩할 함수와 반복할 특정시간, 반복유무 설정
		TimerHandle, this, &ACEnemyFactory::MakeEnemy, SpawnTime, true
	);

}

// ACEnemyFactory.cpp EndPlay()
void ACEnemyFactory::EndPlay(const EEndPlayReason::Type EndPlayReason)
{
	Super::EndPlay(EndPlayReason);

	// EndPlay() 호출시 TimerHandle Valid인지 확인하고 Active 상태라면 해제
	if (TimerHandle.IsValid() and GetWorld()->GetTimerManager().IsTimerActive(TimerHandle))
		GetWorld()->GetTimerManager().ClearTimer(TimerHandle);

}
```

#### 결과 화면
![alt text](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzFfMjU3/MDAxNzM4Mjg5OTYzNzk4.j-cHCyiP9dFmILnCA4S7I5U4ycVskqaSSSl_C8aN_GUg.NgNp1xNRWL0UEdrYWmgnxl_mCf8f2pKH4x3Zpkbfaq0g.GIF/mp4_to_gif_(2).gif?type=w966)
> 위에 결과와 다르지 않지만, Timer 사용법을 익힐 수 있었던 예제

## Github
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/CPP_Shooting2D)
