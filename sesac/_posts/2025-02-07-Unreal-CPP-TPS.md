---
layout: post
description: > 
  적 생성, FSM, 적 피격 및 사망 처리
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.02.07 Unreal C++ TPS
![image](https://cdn.htmr.kr/uploads/news/_thumbnail/642e2b3a64553.jpg)

<!--more-->
* toc 
{:toc .large-only}

## 적(Enemy) Class
### 적(Enemy) SkeletalMesh 적용
```java
UCLASS()
class TPSEXAMPLE_API ACEnemy : public ACharacter
{
	GENERATED_BODY()

	// ... 생략

private:
	void InitializeCharacter();

};
```
```java
void ACEnemy::InitializeCharacter()
{
	// Mesh에 SK_Mannequin 로드 후 설정
	ConstructorHelpers::FObjectFinder<USkeletalMesh> mesh(L"/Script/Engine.SkeletalMesh'/Game/PJS/Characters/Meshes/SK_Mannequin.SK_Mannequin'");
	if (mesh.Succeeded()) // 성공이라면
		GetMesh()->SetSkeletalMesh(mesh.Object);

	// 위치값과 회전값 반영
	GetMesh()->SetRelativeLocationAndRotation(FVector(0, 0, -90), FRotator(0, -90, 0));
}
```
```java
ACEnemy::ACEnemy()
{
	PrimaryActorTick.bCanEverTick = true;

	InitializeCharacter();
}
```
> [플레이어](https://likepint.github.io/sesac/2025-02-03-Unreal-CPP-TPS/#%ED%94%8C%EB%A0%88%EC%9D%B4%EC%96%B4-%EC%83%9D%EC%84%B1-%EA%B2%B0%EA%B3%BC-%ED%99%94%EB%A9%B4)와 동일한 메시를 사용하기에 플레이어의 구조를 그대로 사용

### 적(Enemy) 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMTA5/MDAxNzM4ODkyMzI1MjE5.ghr5mK-KJh1jC7rI7AjaptO8D42fVBdIIeDhdXwsbTog.YkHuLo5wgS5uAUqLVP8Xev12I4JCdrhLlP6Nx71ZKZAg.PNG/image.png?type=w800)

- ### 유한 상태 기계 (FSM, Finite State Machine)
> 주의할 점 : 프레임마다 불려야할 함수인지 아닌지
>> 많이 쓰는 상태 : 대기(Idle), 이동(Move), 공격(Attack) \
>> 상태에서 다른 상태로 이동 할 수 있냐 없냐를 정의 \
>> 예외 상태도 존재하며 대체로 피격 또는 사망 (AnyState)

#### FSMComponent 생성
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMTY5/MDAxNzM4ODkyODMyNDk4.iJwuoNGA6jvH4Lro0OoCi3wDvsoLRTOHDU199os8mRUg.vghwYj-JLjpYwzxuuUJJA8dBPeLujTCrW2wLpxcUb6Mg.PNG/image.png?type=w800)
> ActorComponent를 상속받아 생성

#### 행동 정의 (UENUM)
```java
// 사용할 상태 정의
UENUM ( BlueprintType )
enum class EEnemyState : uint8
{
	Idle = 0, Move, Action, Damaged, Dead, MAX
};

UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class TPSEXAMPLE_API UCEnemyFSMComponent : public UActorComponent
{
	GENERATED_BODY()

	// ... 생략

};
```
> UCLASS 상단에 UENUM을 활용하여 필요한 상태 정의 \
> BlueprintType 키워드를 사용하여 블루프린트에서 확인 가능하도록 설정

```java
Idle = 0 UMETA ( DisplayName = "대기" )
```
> 해당 키워드를 사용하여 블루프린트에서 "대기"로 보이도록 설정 가능

#### 상태에 관련된 변수와 함수
```java
public:
	// 상태 변수
	UPROPERTY(BlueprintReadOnly, VisibleAnywhere, Category = "FSM")
	EEnemyState mState = EEnemyState::Idle;

	/* 각 상태에 대한 함수 */
	// 대기 상태
	void IdleState();
	// 이동 상태
	void MoveState();
	// 공격 상태
	void AttackState();
	// 피격 상태
	void DamagedState();
	// 사망 상태
	void DeadState();
```
```java
void UCEnemyFSMComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	switch (mState)
	{
		case EEnemyState::Idle:
		{
			IdleState();

			break;
		}
		case EEnemyState::Move:
		{
			MoveState();

			break;
		}
		case EEnemyState::Attack:
		{
			AttackState();

			break;
		}
		case EEnemyState::Damaged:
		{
			DamagedState();

			break;
		}
		case EEnemyState::Dead:
		{
			DeadState();

			break;
		}
	}
}
```
> ⁉️ if문 보다는 switch문을 추천

```java
if (mState == EEnemyState::Idle)
{

}
else if (mState == EEnemyState::Idle)
{

}
```
> ⁉️ if문의 경우 이렇게 작성해도 VS에서 오류로 판별하지 않음

#### 적(Enemy) 캐릭터에 FSMComponent 생성
```java
UCLASS()
class TPSEXAMPLE_API ACEnemy : public ACharacter
{
	GENERATED_BODY()

public:
	UPROPERTY(BlueprintReadOnly, VisibleAnywhere, Category = "FSMComponent")
	class UCEnemyFSMComponent* FSM;

	// ... 생략
}
```
```java
#include "Components/CEnemyFSMComponent.h"

void ACEnemy::InitializeCharacter()
{
	// ... 생략

	// EnemyFSM 컴포넌트 추가
	FSM = CreateDefaultSubobject<UCEnemyFSMComponent>("FSM");
}
```
##### FSMComponent 생성 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMTE5/MDAxNzM4ODk1MTQ0Mjc2.Luae5B11S0n6sngPbhwMCXji70J0ZZORYKEua_RzLBAg.hyaR4jhRQfN7ii5NtYtDXZu27TpmG22TZrFgRMH3XRIg.PNG/image.png?type=w800)

#### EnemyState DebugMessage 출력
```java
void UCEnemyFSMComponent::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);

	// 실행창에 상태 메세지 출력
	FString msg = UEnum::GetValueAsString(mState);
	GEngine->AddOnScreenDebugMessage(0, 1, FColor::Red, msg);
}
```

##### DebugMessage 출력 결과 화면
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMzkg/MDAxNzM4ODk1NDMwMTcw.-OW2hBsTJWAd3BkR954dnJRhq9iEScqWMM_0hXQCsLwg.dP-ZFuNWwSdHSzyZYj5efwc7qn40B_m-LyoIot3fKFcg.PNG/image.png?type=w800)

#### IdleState()
```java
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class TPSEXAMPLE_API UCEnemyFSMComponent : public UActorComponent
{
	GENERATED_BODY()

	// ... 생략
		
public:
	// 대기 상태에서 쓸 변수
	// 경과 시간
	UPROPERTY(EditAnywhere , Category = "FSM")
	float CurrentTime = 0;
	// 대기 시간
	UPROPERTY(EditAnywhere , Category = "FSM")
	float IdleDelayTime = 2;

};
```
```java
void UCEnemyFSMComponent::IdleState()
{
	// 시간이 흐르도록 설정
	CurrentTime += GetWorld()->DeltaTimeSeconds;

	// 만약 경과 시간이 대기시간을 초과했다면
	if (CurrentTime >= IdleDelayTime)
	{
		// 이동 상태로 전환
		mState = EEnemyState::Move;

		// 경과 시간 초기화
		CurrentTime = 0;
	}
}
```
##### IdleState -> MoveState 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMjM2/MDAxNzM4OTAzMTkwODY3.718xYV5e5czpJuDNBe8u3nfLIfeVACesuZ8VWVpNhgEg.3NJYA0XBknY-Tssfii5i7sSc9hPsvJAFBEOu14xOLZUg.GIF/EnemyMoveState.gif?type=w966)

#### MoveState()
```java
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class TPSEXAMPLE_API UCEnemyFSMComponent : public UActorComponent
{
	GENERATED_BODY()

	// ... 생략

public:

	// 생략

	// 타겟
	UPROPERTY(VisibleAnywhere, Category = "FSM")
	class ACTPSCharacter* Target;

	// 소유하고 있는 오너
	UPROPERTY()
	class ACEnemy* me;

};
```
```java
#include "Kismet/GameplayStatics.h"
#include "Characters/CTPSCharacter.h"

void UCEnemyFSMComponent::BeginPlay()
{
	Super::BeginPlay();

	// 월드에서 ATPSCharacter 찾기
	if (auto actor = UGameplayStatics::GetActorOfClass(GetWorld(), ACTPSCharacter::StaticClass()))
	{
		// ACTPSCharacter 캐스팅
		Target = Cast<ACTPSCharacter>(actor);

		// 오너 불러오기
		me = Cast<ACEnemy>(GetOwner());
	}
}

// ... 생략

void UCEnemyFSMComponent::MoveState()
{
	// 목적지 설정
	FVector dest = Target->GetActorLocation();

	// 방향 설정
	FVector dir = dest - me->GetActorLocation();

	// dir 방향으로 캐릭터 이동
	me->AddMovementInput(dir.GetSafeNormal());
}
```
##### MoveState() 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMjM2/MDAxNzM4OTAzMTkwODY3.718xYV5e5czpJuDNBe8u3nfLIfeVACesuZ8VWVpNhgEg.3NJYA0XBknY-Tssfii5i7sSc9hPsvJAFBEOu14xOLZUg.GIF/EnemyMoveState.gif?type=w966)

##### 적캐릭터가 플레이어를 보면서 이동
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMTY2/MDAxNzM4OTAyOTc3MDgz.atpRQSQGfIIAtPFv0Gm64N_fsUF4P4QfD-MoUI5h934g.U3n-Z6P1jw2A6NCOl8OdDqa1ugi9YofQOaN6-P1o9-kg.PNG/image.png?type=w800)
> "CharacterMovementComponent"에서 \
> "Orient Rotation To Movement" 해당 옵션을 체크해주면 이동 방향으로 캐릭터를 회전하여 이동

###### 플레이어 방향으로 회전하여 이동 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMTM4/MDAxNzM4OTAzNTUxMTkz.PGyOeQiEfHuVwptVypXI_CFi5WDwnikcnVtRw-kgy0wg.PZ7JI_qfCwsnkqQ1tBj72XWOEPwFprKwnrclyim0_E8g.GIF/EnemyMoveStateRotation.gif?type=w966)

#### AttackState()
```java
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class TPSEXAMPLE_API UCEnemyFSMComponent : public UActorComponent
{
	GENERATED_BODY()

	// ... 생략

public:
	// 공격 범위
	UPROPERTY(VisibleAnywhere, Category = "FSM")
	float AttackRange = 150;

	// 공격 대기 시간
	UPROPERTY(VisibleAnywhere, Category = "FSM")
	float AttackDelayTime = 2;

};
```
```java
void UCEnemyFSMComponent::MoveState()
{
	// ... 생략

	// 타겟과 거리를 체크해서 AttackRange 안으로 들어오면 공격 상태로 전환
	// 거리 체크
	if (dir.Size() <= AttackRange)
		AttackState();
}

void UCEnemyFSMComponent::AttackState()
{
	// 일정 시간에 한번씩 공격
	// 시간이 흐르다가
	CurrentTime += GetWorld()->DeltaTimeSeconds;

	// 공격시간이 되면
	if (CurrentTime >= AttackDelayTime)
	{
		// 공격 실행
		PRINT_LOG(TEXT("Attack~!~!~!"));

		// 결과 시간 초기화
		CurrentTime = 0;
	}
}
```

##### AttackState() 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMjIz/MDAxNzM4OTA0ODU3MTUw.sreKwOYSOz8Qyg53iPvrl8F1arnNbDPddTuwJccOcIYg.Y3mETsn-BGtjyJJR3-N3pVuRgYylR7JMQuGJQ5ayBMIg.GIF/EnemyAttackState.gif?type=w966)

##### AttackState -> MoveState
```java
void UCEnemyFSMComponent::AttackState()
{
	// ... 생략

	// 타겟이 공격범위를 벗어나면 이동상태로 전환
	// 타겟과의 거리
	float dist = FVector::Distance(Target->GetActorLocation(), me->GetActorLocation());

	// 타겟과의 거리가 공격 범위를 벗어나면
	if (dist > AttackRange)
		mState = EEnemyState::Move;
}
```

###### AttackState -> MoveState 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMjMx/MDAxNzM4OTA1MDg4NTky.lxZYSpri4SHSqWu9Ocu1BvZ-0k43yPnot1aFVcauXSMg.V30W9y4HbFDvZUQKdhARjgPIlSE58JGKZkupryO5OtYg.GIF/AttackToMove.gif?type=w966)

#### DamagedState
##### Enemy HP 및 피격 시간 설정
```java
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class TPSEXAMPLE_API UCEnemyFSMComponent : public UActorComponent
{
	GENERATED_BODY()

	// ... 생략

public:
	// ... 생략

	// 체력
	UPROPERTY(BlueprintReadWrite, EditDefaultsOnly, Category = "FSM")
	int32 hp = 3;

	// 피격 대기 시간
	UPROPERTY(VisibleAnywhere, Category = "FSM")
	float DamageDelayTime = 2;
};
```
```java
void UCEnemyFSMComponent::DamagedState()
{
	// 시간이 흐르다가
	CurrentTime += GetWorld()->DeltaTimeSeconds;

	// 경과 시간이 대기 시간을 초과하면
	if (CurrentTime >= DamageDelayTime)
	{
		// 대기 상태로 전환
		mState = EEnemyState::Idle;

		// 경과 시간 초기화
		CurrentTime = 0;
	}
}
```
```java
void UCEnemyFSMComponent::OnDamageProcess()
{
	// 체력 감소
	hp -= 1;

	// 체력이 남아 있는지 체크
	if (hp > 0)
		 mState = EEnemyState::Damaged;
	else mState = EEnemyState::Dead;
}
```

#### DeadState()
> DeadState() 호출 시 캐릭터가 아래로 내려가도록 설정

```java
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class TPSEXAMPLE_API UCEnemyFSMComponent : public UActorComponent
{
	GENERATED_BODY()

	// ... 생략

public:
	// ... 생략

	UPROPERTY(VisibleAnywhere, Category = "FSM")
	float DeadSpeed = 50;
};
```

```java
void UCEnemyFSMComponent::OnDamageProcess()
{
	// 체력 감소
	hp -= 1;

	// 체력이 남아 있는지 체크
	if (hp > 0)
		 mState = EEnemyState::Damaged;
	else
	{
		mState = EEnemyState::Dead;

		// 캡슐의 충돌체 비활성화
		me->GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	}
}
```
> 캡슐의 충돌체를 비활성화 하지 않으면 캐릭터가 밑으로 내려가지 않음

##### DeadState() 호출 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMjg2/MDAxNzM4OTA3MTMzMTEy.O5iEXI9p0kWw8Z37jM7NZQRjhJkwpxsq_An_i08mUzsg.CduPcdPv8GtC5NOgOQnuIZ_nWweVicIaPUcSI3_5vnEg.GIF/DeadState.gif?type=w966)

### Enemy 피격
```java
UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class TPSEXAMPLE_API UCEnemyFSMComponent : public UActorComponent
{
	GENERATED_BODY()

	// ... 생략
		
public:
	// ... 생략

	// 피격 알림 이벤트 함수
	void OnDamageProcess();
}
```
```java
void ACTPSCharacter::OnFire(const FInputActionValue& InVal)
{
	// Rifle
	if (bRifle) // {  }
	// Sniper
	else
	{
		// ... 생략

		// LineTrace가 충돌했을 때
		if (bHit)
		{
			// ... 생략

			// 부딪힌 대상이 적인 체크
			auto enemy = info.GetActor()->GetDefaultSubobjectByName("FSM");

			if (enemy)
			{
				auto enemyFSM = Cast<UCEnemyFSMComponent>(enemy);
				enemyFSM->OnDamageProcess();
			}
		}
	}
}
```
```java
void UCEnemyFSMComponent::OnDamageProcess()
{
	me->Destroy();
}
```

#### Destroy() 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDdfMzIg/MDAxNzM4OTA1ODIyNzE0.UoyWpibU-HRKmd7xoq2JoXCS8JYpjrd4bAISOKA-jTsg.Ginjh6gCtFLfDIgE-tee7z6K2uu5kNUciTHSEMzFOr8g.GIF/EnemyDamaged.gif?type=w966)



## Github Address (TPSExample Repository)
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/SeSAC_TPSExample)
