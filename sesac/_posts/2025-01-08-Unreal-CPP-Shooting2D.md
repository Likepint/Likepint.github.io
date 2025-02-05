---
layout: post
description: > 
  UE_LOG, OnScreenDebugMessage 출력
accent_image: 
excerpt_separator: <!--more-->
sitemap: false
---

# 2025.01.08 Unreal C++ Shooting2D
![image](https://cdn.htmr.kr/uploads/news/_thumbnail/642e2b3a64553.jpg)

<!--more-->
* toc 
{:toc .large-only}

## UE_LOG를 사용하여 Hello World 출력

```c++
// define 원형
#define UE_LOG(CategoryName, Verbosity, Format, ...) \
	UE_PRIVATE_LOG(PREPROCESSOR_NOTHING, constexpr, CategoryName, Verbosity, Format, ##__VA_ARGS__)

UE_LOG(<LOG_CATEGORY>, <VERBOSITY_LEVEL>, TEXT("SENTENCE"));

```

![](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/8f35c931-bc2a-4ca6-a929-fa1c63d1f9b0/outputlogwindow.png)
```c++
UE_LOG(LogTemp, Warning, TEXT("Hello World"));

``` 

### Log Verbosity levels

|    열거형   |                                                                   설명                                                                  |
|:-----------:|:---------------------------------------------------------------------------------------------------------------------------------------:|
|    Fatal    |                   로깅이 비활성화된 경우에도 항상 치명적인 오류를 콘솔 및 로그 파일에 출력한 후 크래시를 발생시킵니다.                  |
|    Error    | 오류를 콘솔 및 로그 파일에 출력합니다. 커맨드릿과 에디터가 오류를 수집하고 보고합니다. 오류 메시지의 결과로 커맨드릿 실패가 발생합니다. |
|   Warning   |                          경고를 콘솔 및 로그 파일에 출력합니다. 커맨드릿과 에디터가 경고를 수집하고 보고합니다.                         |
|   Display   |                                                 메시지를 콘솔 및 로그 파일에 출력합니다.                                                |
|     Log     |                                      메시지를 로그 파일에는 출력하지만 콘솔에는 출력하지 않습니다.                                      |
|   Verbose   |         해당 카테고리에 대해 상세 로깅이 활성화된 경우 상세 메시지를 로그 파일에 출력합니다. 일반적으로 상세 로깅에 사용됩니다.         |
| VeryVerbose |     상세 메시지를 로그 파일에 출력합니다. VeryVerbose 로깅이 활성화된 경우 이는 다른 경우에 스팸으로 출력될 상세 로깅에 사용됩니다.     |



## DebugMessage를 사용하여 Hello World 출력

```c++
// 함수 원형
void UEngine::AddOnScreenDebugMessage
(
    int32 Key,
    float TimeToDisplay,
    FColor DisplayColor,
    const FString& DebugMessage,
    bool bNewerOnTop,
    const FVector2D& TextScale
)

GEngine->AddOnScreenDebugMessage(-1, 5.f, FColor::White, TEXT("This is an Example on-screen debug message."));

```
![](https://d1iv7db44yhgxn.cloudfront.net/documentation/images/8fdd74a2-3b9c-4d9e-a2c5-465f203ea941/printscreenmessage.png)

 - 첫 번째 파라미터 key 는 고유한 인티저 키를 사용하여 동일한 메시지가 여러 번 추가되지 않도록 방지합니다.
 - 두 번째 파라미터 TimeToDisplay 는 플로트를 사용하여 메시지가 사라지기 전 메시지를 표시할 시간(초)을 지정합니다.
 - 세 번째 파라미터 DisplayColor 는 텍스트를 표시할 컬러를 취합니다.
 - 네 번째 파라미터 DebugMessage 는 표시할 메시지입니다. 로그와 비슷하게 화면 디버그 메시지에서 포맷 지정자 및 변수를 사용할 수 있습니다.

## Github
[![Github](https://mblogthumb-phinf.pstatic.net/MjAyNTAxMzBfMTA1/MDAxNzM4MjAxMDQyMjQ3.oQfi6oNsgKIzoWlLEuMJAAmVWoKfAaSkD9Iz7jGwtzQg.LT6UWR0c581WX7Z14Iw89jOXcWYN13qQa2x7sb7zX1Yg.JPEG/Github.jpg?type=w800)](https://github.com/Likepint/CPP_Shooting2D)
