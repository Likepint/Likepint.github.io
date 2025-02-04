---
layout: post
description: > 
  Enhanced Input 설정 
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.02.04 Unreal C++ TPS
- ## Enhanced Input 설정
![image](https://i.ytimg.com/vi/CYiHNbAIp4s/maxresdefault.jpg)

<!--more-->
* toc 
{:toc .large-only}

- ### 마우스(Mouse) 회전 구현
#### IA_VerticalLook
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMjQx/MDAxNzM4NjMzNjY0MjA5.00y-7txV0M75rtFxQk_eozOWLdH9mx-V7JpVdrUICdgg.-51i5HtmyxT44Q1H-P2-A2eH9znKxxj9MgqC2BE5xJ0g.PNG/image.png?type=w800)
#### IA_HorizontalLook
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMjU0/MDAxNzM4NjMzNjg3Mjc2.LI5e-PBzdaMv4nTEAkxIV67wAodeNX_MLSTpC3xbf1Ig.WjwKD8tvrblFQ5zVxW3tWOvBkLybeYi_Xm-pIx_MvsAg.PNG/image.png?type=w800)
#### Input Mapping Context
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMjY4/MDAxNzM4NjMzNjMzMTQ5._91UZSu8HeHuHgNhV2I39nyYMR-IZqStYtc5uvTiKRcg.GRiJcHK0CSskFKmnsVghwiEv-4PLwuKYC8HMUzea3Fog.PNG/image.png?type=w800)

#### bUsePawnControlRotation
```java
void ACTPSCharacter::InitializeCharacter()
{
	// ... 생략

	// bUseControllerRotationYaw 설정
	bUseControllerRotationYaw = true;

	// ... 생략

	// UsePawnControlRotation 설정
	SpringArm->bUsePawnControlRotation = true;

	// ... 생략

	// UsePawnControlRotation 설정
	Camera->bUsePawnControlRotation = false;

}
```

#### BeginPlay() 설정
```java
void ACTPSCharacter::BeginPlay()
{
	Super::BeginPlay();
	
	if (APlayerController* controller = Cast<APlayerController>(GetController()))
	{	// Get local player subsystem
		if (UEnhancedInputLocalPlayerSubsystem* Subsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(controller->GetLocalPlayer()))
			// Add input context
			Subsystem->AddMappingContext(IMC_TPS, 0);
	}
}
```

#### OnVerticalLook()
```java
void ACTPSCharacter::OnVerticalLook(const FInputActionValue& InVal)
{
	float value = InVal.Get<float>();

	AddControllerPitchInput(value);
	
}
```

#### OnHorizontalLook()
```java
void ACTPSCharacter::OnHorizontalLook(const FInputActionValue& InVal)
{
	float value = InVal.Get<float>();

	AddControllerYawInput(value);

}
```

#### 함수 바인딩
```java
void ACTPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		EnhancedInputComponent->BindAction(IA_VerticalLook, ETriggerEvent::Triggered, this, &ACTPSCharacter::OnVerticalLook);
		EnhancedInputComponent->BindAction(IA_HorizontalLook, ETriggerEvent::Triggered, this, &ACTPSCharacter::OnHorizontalLook);

	}
}
```

#### 마우스(Mouse) 회전 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMTYg/MDAxNzM4NjQyODY4OTE5.qcLSJM37YBoTJoTwo90mqIrOONXCe6CLHs4i__QqGQog.iHq0u_A-I158HwNR89h6LnQ01uy6hN91lwLM-3AtgnEg.GIF/Look.gif?type=w966)

- ### 이동(Movement) 구현

#### IA_Movement
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMTEx/MDAxNzM4NjQyNzc3MzIz.CGk5dvljaXBLvq3eQ4pf5wpNw7LdYb3iS4G9p0hc1PEg.ZHZBqSTWCXmJotsYuXfjIr2lsau6b_jjr6oZP0QH5jYg.PNG/image.png?type=w800)

#### Input Mapping Context
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMjQ1/MDAxNzM4NjQyODUzNjg4.GTAI2saVF9M8yAaSVOiWVbb_6z1amAgcwUCJEG9GSR4g.e5A9KJGbALXMHLH7lyAz7XKlPqxbfkAcfhUXi-BnVNQg.PNG/image.png?type=w800)
> 캐릭터의 정면을 기준으로, W는 X축 정면, D는 Y축의 정면(우측방향)이므로 **Swizzle** 시켜서 사용

#### OnMovement()
```java
void ACTPSCharacter::OnMovement(const FInputActionValue& InVal)
{
	// Forward
	AddMovementInput(GetActorForwardVector(), InVal.Get<FVector>().X);

	// Right
	AddMovementInput(GetActorRightVector(), InVal.Get<FVector>().Y);

}
```

#### 함수 바인딩
```java
void ACTPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		// ... 생략

		EnhancedInputComponent->BindAction(IA_Movement, ETriggerEvent::Triggered, this, &ACTPSCharacter::OnMovement);

	}
}
```

#### 이동(Movement) 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMjk3/MDAxNzM4NjQzNjgzNjk2.zt-Kg403P_QKc4ODv4F6-96vOg_aXe5jV5N6eSEoiKMg.CyHOxGkm0Mlx_EjVb2Odes9nDGPf3-byJ5Y9I-68BLwg.GIF/Movement.gif?type=w966)

- ### 점프(Jump) 구현
#### IA_JumpAction
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfMjAg/MDAxNzM4NjQ4MjU1MzAx.SeRXdwE9qE5zVzU1oQUJN5PAxQLju2EKh3Jhsfb87esg.HAblJgL8f6ELIwtiTFhFHL-6WNRkJnIl5YoiBcA95CEg.PNG/image.png?type=w800)

#### Input Mapping Context
![image](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfNDQg/MDAxNzM4NjQ4MjI5MTEw.zpWOWBM7tavDLcz-F9NkupI041462YPLbU_v0wr2YlYg.JBABmJPLI48f9gFq5-4bDEkRK_SPhW7t8B9a9BAgTdgg.PNG/image.png?type=w800)

#### OnJumpAction();
```java
void ACTPSCharacter::OnJumpAction(const FInputActionValue& InVal)
{
	Jump();

}
```

#### 함수 바인딩
```java
void ACTPSCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	if (UEnhancedInputComponent* EnhancedInputComponent = Cast<UEnhancedInputComponent>(PlayerInputComponent))
	{
		// ... 생략

		EnhancedInputComponent->BindAction(IA_JumpAction, ETriggerEvent::Started, this, &ACTPSCharacter::OnJumpAction);

	}
}
```

#### 점프(Jump) 결과 화면
![gif](https://mblogthumb-phinf.pstatic.net/MjAyNTAyMDRfNDgg/MDAxNzM4NjQ4ODgwMDQy.FfnkLLVlmx54m4xtKE03kMGrr8Tf1Kr1DNMHnrk7PBQg.0HCGOwW39e7P2CoX0Iw3UpaaUO1r397C9K2s7PnY5fEg.GIF/Jump.gif?type=w966)

## Github Address (TPSExample Repository)
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/SeSAC_TPSExample)
