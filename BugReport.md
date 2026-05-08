# Bug Report — AVaOut: Avatar Out

---

## UI-001

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | M-001 |
| **보고일** | 2026-05-03 |
| **보고자** | jch1999 |
| **우선순위** | P1-High |
| **영역** | 멀티플레이 |
| **카테고리** | UI |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

메인 메뉴에서 CreateGame 버튼 클릭 시 방 생성 다이얼로그가 출력되지 않음

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 호스트 단독 실행 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 에디터에서 `LV_MainMenu` 레벨을 PIE로 실행
2. 메인 메뉴 화면에서 **Create Game** 버튼 클릭

### 기대 결과

방 이름 입력 등 룸 생성 정보를 입력하는 `UAO_HostDialogWidget` 다이얼로그가 화면에 출력된다.

### 실제 결과

다이얼로그가 전혀 출력되지 않는다. 버튼 클릭 자체는 처리되며 아래 로그가 정상 출력된다.

```
LogJSH: UAO_MainMenuWidget::OnClicked_Host : Host clicked
LogJSH: UAO_HostDialogWidget::ApplyModalInput : ApplyModalInput: UIOnly input mode set
LogJSH: UAO_HostDialogWidget::UpdateCreateButtonState : UpdateCreateButtonState: Name="" Enable=0
LogJSH: UAO_HostDialogWidget::NativeConstruct : HostDialog shown (mouse-only modal)
LogJSH: UAO_MainMenuWidget::OnClicked_Host : OnClicked_Host: HostDialog popup displayed
```

로그상으로는 위젯이 생성·추가되었으나 화면에 보이지 않는다.

### 원인 분석

**파일:** `Source/AO/Private/UI/Widget/AO_MainMenuWidget.cpp`  
**함수:** `UAO_MainMenuWidget::OnClicked_Host()` (16번째 줄)

`UAO_HostDialogWidget`을 생성하고 `AddToViewport()` 한 직후 `SetVisibility(ESlateVisibility::Hidden)`을 호출하고 있었다. 위젯은 정상 생성되어 뷰포트에 추가되지만 즉시 숨김 처리되어 출력되지 않았다.

Widget Reflector로 추적 시 해당 위젯의 Visibility가 `Hidden` 상태인 것으로 확인되었다.

### 수정 내용

**파일:** `Source/AO/Private/UI/Widget/AO_MainMenuWidget.cpp`, 29번째 줄

```cpp
// 수정 전
Dialog->SetVisibility(ESlateVisibility::Hidden);

// 수정 후
Dialog->SetVisibility(ESlateVisibility::Visible);
```

### 검증 결과

수정 후 컴파일 및 PIE 실행 시 Create Game 버튼 클릭에 정상적으로 방 생성 다이얼로그가 출력되는 것을 확인.

---

## CHAR-001

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | S-002 |
| **보고일** | 2026-05-03 |
| **보고자** | jch1999 |
| **우선순위** | P2-Medium |
| **영역** | 싱글플레이 |
| **카테고리** | 기능 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

웅크리기(Crouch) 키 입력 시 캐릭터 반응 없음

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 싱글플레이 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 메인 메뉴 레벨 진입
2. 튜토리얼 버튼을 눌러 튜토리얼 레벨 진입
3. 웅크리기 키 입력
4. 캐릭터 키 감소 애니메이션 확인
5. 캡슐 콜리전 높이 감소 확인
6. 좁은 공간 통과 확인

### 기대 결과

웅크리기 즉시 애니메이션 전환, 콜리전 높이 축소, 서 있을 공간 없으면 자동 유지

### 실제 결과

튜토리얼 맵 진입은 정상이나 웅크리기 키 입력에 대한 피드백이 발생하지 않는다.

### 원인 분석

**파일:** `Source/AO/Private/Character/AO_PlayerCharacter.cpp`  
**함수:** `AAO_PlayerCharacter::SetupPlayerInputComponent()`

`HandleCrouch()` 함수는 539번째 줄에 정상 구현되어 있으나, `SetupPlayerInputComponent()`의 바인딩 목록에서 `IA_Crouch`에 대한 연결이 누락되어 있었다.

```cpp
// 누락된 바인딩 목록
EIC->BindAction(IA_Move, ETriggerEvent::Triggered, this, &AAO_PlayerCharacter::Move);
EIC->BindAction(IA_Look, ETriggerEvent::Triggered, this, &AAO_PlayerCharacter::Look);
EIC->BindAction(IA_Jump, ETriggerEvent::Started,   this, &AAO_PlayerCharacter::StartJump);
EIC->BindAction(IA_Jump, ETriggerEvent::Triggered, this, &AAO_PlayerCharacter::TriggerJump);
EIC->BindAction(IA_Walk, ETriggerEvent::Started,   this, &AAO_PlayerCharacter::HandleWalk);
// IA_Crouch 바인딩 없음
```

### 수정 내용

**파일:** `Source/AO/Private/Character/AO_PlayerCharacter.cpp`

```cpp
// 추가
EIC->BindAction(IA_Crouch, ETriggerEvent::Started, this, &AAO_PlayerCharacter::HandleCrouch);
```

### 검증 결과

수정 후 정상 동작 확인

---

## CHAR-002

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | S-004, S-011 |
| **보고일** | 2026-05-03 |
| **보고자** | jch1999 |
| **우선순위** | P2-Medium |
| **영역** | 싱글플레이 |
| **카테고리** | 기능 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

스프린트 키 입력 시 속도 변화 없음

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 싱글플레이 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 스프린트 키 홀드
2. SprintSpeed 적용 및 스태미나 2f/초 감소 확인
3. 스태미나 0 도달 시 스프린트 불가 확인
4. 정지 후 자연 회복 확인

### 기대 결과

홀드 시 빠른 이동 속도 및 스태미나 감소, 소진 시 스프린트 불가, 정지 후 정상 회복

### 실제 결과

스프린트 키를 입력해도 속도 변화를 체감하지 못한다. 걷기 키 입력 시에는 토글 동작, 속도 확연한 감소가 정상 확인되어 달리기가 기본 설정일 가능성이 있다.

### 원인 분석

**파일:** `Source/AO/Private/Character/GAS/AO_PlayerCharacter_AttributeSet.cpp`  
**함수:** `UAO_PlayerCharacter_AttributeSet::HandleStaminaLockout()`

락아웃 적용·해제 조건이 반전되어 있었다. 스태미나가 `0.f` 이상이면(즉 사실상 항상) `Status.Lockout.Stamina` 태그가 추가되어 `GA_Sprint`의 `ActivationBlockedTags`와 맞물려 스프린트가 영구 차단되고, 해제 분기(`else if`)는 스태미나가 `Threshold` 이하일 때 실행되도록 되어 있어 절대 도달할 수 없었다. 또한 `PreAttributeChange()`에서 스태미나를 `[0, MaxStamina]`로 클램프하므로 음수는 발생하지 않아 해제 조건은 더욱 성립 불가했다.

```cpp
// 버그 코드
if (NewStamina >= 0.f)   // 사실상 항상 true → 락아웃 태그 추가
{
    if (!ASC->HasMatchingGameplayTag(LockoutTag))
    {
        ASC->AddLooseGameplayTag(LockoutTag);
        ASC->CancelAbilities(&SprintTag);
    }
}
else if (ASC->HasMatchingGameplayTag(LockoutTag) && NewStamina <= Threshold)  // 절대 도달 불가
{
    ASC->RemoveLooseGameplayTag(LockoutTag);
}
```

### 수정 내용

**파일:** `Source/AO/Private/Character/GAS/AO_PlayerCharacter_AttributeSet.cpp`

```cpp
// 수정 후
if (NewStamina <= 0.f)   // 스태미나 소진 → 락아웃 추가
{
    if (!ASC->HasMatchingGameplayTag(LockoutTag))
    {
        ASC->AddLooseGameplayTag(LockoutTag);
        ASC->CancelAbilities(&SprintTag);
    }
}
else if (ASC->HasMatchingGameplayTag(LockoutTag) && NewStamina >= Threshold)  // 회복 → 락아웃 해제
{
    ASC->RemoveLooseGameplayTag(LockoutTag);
}
```

### 검증 결과

수정 후 정상 동작 확인

---

## CHAR-003

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | S-041 |
| **보고일** | 2026-05-03 |
| **보고자** | jch1999 |
| **우선순위** | P2-Medium |
| **영역** | 싱글플레이 |
| **카테고리** | 기능 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

커스터마이징 저장 후 본 레벨 복귀 시 변경 사항이 적용되지 않음

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 싱글플레이 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 커스터마이즈용 아바타 액터와 상호작용
2. 의상 색상 변경 등 외형 수정
3. 저장 버튼 누르고 본 레벨로 복귀
4. 캐릭터 외형 확인

### 기대 결과

커스터마이징 레벨에서 수정된 외형 정보가 정상 저장되어 본 레벨 복귀 후에도 적용된다.

### 실제 결과

수정 전 외형을 그대로 유지하고 있다. 변경 사항이 저장되지 않는 것으로 보인다.

### 원인 분석

`SaveCustomizingData()`가 `PlayerState`에만 데이터를 저장하고 현재 세션의 `CustomizingComponent`를 갱신하지 않았다. `ServerRPC_SetCharacterCustomizingData_Implementation` 내부에서 Pawn의 `CustomizingComponent`를 찾아 `ServerRPC_ChangeCustomizing()`을 추가로 호출하는 방식으로 수정.

### 수정 내용

**파일:** `Source/AO/Private/Player/PlayerState/AO_PlayerState.cpp`

```cpp
void AAO_PlayerState::ServerRPC_SetCharacterCustomizingData_Implementation(
    const FCustomizingData& CustomizingData)
{
    CharacterCustomizingData = CustomizingData;
    if (APawn* Pawn = GetPawn())
    {
        if (UAO_CustomizingComponent* CustomComp =
                Pawn->FindComponentByClass<UAO_CustomizingComponent>())
        {
            CustomComp->ServerRPC_ChangeCustomizing(CharacterCustomizingData);
        }
    }
}
```

### 검증 결과

수정 후 커스터마이징 저장 시 현재 세션 캐릭터 메쉬에 즉시 반영됨을 확인.

---

## CHAR-004

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | S-003 |
| **보고일** | 2026-05-03 |
| **보고자** | jch1999 |
| **우선순위** | P1-High |
| **영역** | 싱글플레이 |
| **카테고리** | 기능 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

점프 키 입력 시 캐릭터가 점프하지 않음

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 싱글플레이 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 점프 키(Space Bar) 입력
2. `GA_Jump` 발동 및 공중 애니메이션 확인
3. 착지 시 `LandVelocity` 기록
4. 착지 애니메이션 및 사운드 재생 확인

### 기대 결과

스페이스 바 입력 시 캐릭터가 제자리에서 점프하고 공중 및 착지 애니메이션이 재생된다.

### 실제 결과

점프 동작이 작동하지 않는다.

### 원인 분석

**파일:** `Source/AO/Private/Character/AO_PlayerCharacter.cpp`  
**함수:** `AAO_PlayerCharacter::StartJump()`

Traversal 시도 후 점프 어빌리티를 발동하는 태그 문자열에 타이포가 있었다. 어빌리티에 등록된 태그는 `"Ability.Movment.Jump"`이나 호출 시 `"Ability.Movement.Junp"`로 작성되어 태그가 일치하지 않아 어빌리티가 발동되지 않았다.

```cpp
// 버그 코드
AbilitySystemComponent->TryActivateAbilitiesByTag(
    FGameplayTagContainer(FGameplayTag::RequestGameplayTag(FName("Ability.Movement.Junp")))); // ← "Junp"
```

### 수정 내용

**파일:** `Source/AO/Private/Character/AO_PlayerCharacter.cpp`

```cpp
// 수정 전
FName("Ability.Movement.Junp")

// 수정 후
FName("Ability.Movment.Jump")
```

### 검증 결과

수정 후 정상 동작 확인

---

## INV-001

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | S-014 |
| **보고일** | 2026-05-04 |
| **보고자** | jch1999 |
| **우선순위** | P0-Critical |
| **영역** | 싱글플레이 |
| **카테고리** | 크래시 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

인벤토리가 가득 찬 상태에서 아이템 줍기 시도 시 크래시 발생

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 싱글플레이 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 인벤토리가 가득 찬 상태로 게임 진행
2. 월드에 배치된 아이템 근처 이동
3. 아이템 줍기 입력 (인터랙션 키)

### 기대 결과

인벤토리가 가득 찬 경우 픽업 불가 처리 (무시 또는 알림 표시)

### 실제 결과

`EXCEPTION_ACCESS_VIOLATION` 크래시 발생.

```
Unhandled Exception: EXCEPTION_ACCESS_VIOLATION reading address 0x00000000000001f0

UnrealEditor_AO!UAO_InventoryComponent::PickupItem() [AO_InventoryComponent.cpp:129]
UnrealEditor_AO!AAO_MasterItem::Server_HandleInteraction_Implementation() [AO_MasterItem.cpp:178]
UnrealEditor_AO!UGA_Interact_Base::ActivateAbility() [GA_Interact_Base.cpp:49]
UnrealEditor_AO!UAO_GameplayAbility_Interact_Execute::ExecuteInteraction() [AO_GameplayAbility_Interact_Execute.cpp:345]
UnrealEditor_AO!UAO_InteractionComponent::OnInteractPressed() [AO_InteractionComponent.cpp:163]
```

### 원인 분석

**파일:** `Source/AO/Private/Item/invenroty/AO_InventoryComponent.cpp`  
**함수:** `UAO_InventoryComponent::PickupItem()` (128번째 줄)

인벤토리가 가득 찬 상태에서의 교체 로직에서 `AActor* OwnerActor = nullptr`로 선언 후 초기화 없이 바로 역참조하여 `EXCEPTION_ACCESS_VIOLATION` 크래시가 발생했다.

```cpp
// 버그 코드
AActor* OwnerActor = nullptr;
OwnerActor->SomeMethod(); // ← nullptr 역참조
```

### 수정 내용

**파일:** `Source/AO/Private/Item/invenroty/AO_InventoryComponent.cpp`, 128번째 줄

```cpp
// 수정 전
AActor* OwnerActor = nullptr;

// 수정 후
AActor* OwnerActor = GetOwner();
```

### 검증 결과

수정 후 정상 동작 확인

---

## COMBAT-001

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | S-007 |
| **보고일** | 2026-05-06 |
| **보고자** | jch1999 |
| **우선순위** | P1-High |
| **영역** | 호스트 단독 |
| **카테고리** | 기능 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

피격 후 영구 무적 및 스프린트/점프 차단

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 호스트 단독 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 호스트 단독 세션 생성 후 본 게임 맵 진입
2. AI에게 1회 피격
3. 피격 리액션 종료 후 추가 피격 시도
4. 스프린트·점프 입력 시도

### 기대 결과

피격 리액션 몽타주 종료 후 무적 GE가 해제되어 추가 피해를 정상 수신하고, 스프린트·점프가 정상 작동한다.

### 실제 결과

1회 피격 이후 추가 피격이 전혀 적용되지 않으며, 스프린트와 점프가 영구적으로 차단된다.

### 원인 분석

**파일:** `Source/AO/Private/Character/Combat/Ability/AO_GameplayAbility_HitReact.cpp`  
**함수:** `UAO_GameplayAbility_HitReact::ActivateAbility()`

`UAbilityTask_PlayMontageAndWait` 태스크 생성 및 델리게이트 바인딩 후 `ReadyForActivation()`이 호출되지 않아 태스크가 시작되지 않았다. 태스크가 미시작 상태이므로 `OnCompleted`/`OnCancelled` 콜백이 발동되지 않고, `EndAbility()`가 호출되지 않아 피격 시 적용된 무적 GE(`InvulnerableEffectClass`)와 어빌리티 차단 GE(`BlockAbilitiesEffectClass`)가 영구적으로 유지되었다.

### 수정 내용

**파일:** `Source/AO/Private/Character/Combat/Ability/AO_GameplayAbility_HitReact.cpp`

```cpp
// 수정 전 — ReadyForActivation() 누락
MontageTask->OnInterrupted.AddDynamic(this, &UAO_GameplayAbility_HitReact::OnMontageCancelled);

// 수정 후
MontageTask->OnInterrupted.AddDynamic(this, &UAO_GameplayAbility_HitReact::OnMontageCancelled);
MontageTask->ReadyForActivation();
```

### 검증 결과

수정 후 피격 리액션 종료 시 무적 및 어빌리티 차단이 정상 해제되어 추가 피격 수신, 스프린트, 점프가 정상 작동함을 확인.

---

## COMBAT-002

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | S-008 |
| **보고일** | 2026-05-06 |
| **보고자** | jch1999 |
| **우선순위** | P1-High |
| **영역** | 호스트 단독 |
| **카테고리** | 기능 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

체력 0 도달 시 사망 처리 미발동 — 플레이어 조작 지속

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 호스트 단독 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 호스트 단독 세션 생성 후 본 게임 맵 진입
2. AI에게 반복 피격하여 플레이어 체력을 0으로 유도
3. 체력 0 도달 이후 이동·점프 등 조작 시도

### 기대 결과

체력이 0에 도달하면 `OnPlayerDeath`가 브로드캐스트되어 `GA_Death`가 활성화되고, 사망 몽타주 재생 후 래그돌 전환 및 모든 어빌리티가 차단된다.

### 실제 결과

체력이 0이 되어도 사망 처리가 발동되지 않으며 플레이어 캐릭터를 계속 조작할 수 있다.

### 원인 분석

**파일:** `Source/AO/Private/Character/GAS/AO_PlayerCharacter_AttributeSet.cpp`  
**함수:** `UAO_PlayerCharacter_AttributeSet::PostGameplayEffectExecute()` (51번째 줄)

`PreAttributeChange()`에서 체력을 `FMath::Clamp(NewValue, 0.f, MaxHealth)`로 클램프하기 때문에, `PostGameplayEffectExecute()` 시점의 `GetHealth()`는 최솟값이 `0.f`이다. 그러나 사망 분기 조건이 `NewHealth < 0.f`로 작성되어 있어, 체력이 정확히 `0.f`일 때 조건이 성립하지 않아 `OnPlayerDeath.Broadcast()`가 호출되지 않았다.

```cpp
// 문제 코드
const float NewHealth = GetHealth();  // 최솟값 0.f (PreAttributeChange에서 클램프됨)
if (NewHealth < 0.f)                  // 항상 false — 사망 이벤트 미발동
{
    OnPlayerDeath.Broadcast();
}
```

### 수정 내용

**파일:** `Source/AO/Private/Character/GAS/AO_PlayerCharacter_AttributeSet.cpp`, 51번째 줄

```cpp
// 수정 전
if (NewHealth < 0.f)

// 수정 후
if (FMath::IsNearlyZero(NewHealth))
```

### 검증 결과

수정 후 체력 0 도달 시 `GA_Death`가 정상 발동되어 사망 몽타주 재생 및 어빌리티 차단이 확인됨.

---

## TRAIN-001

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | - |
| **보고일** | 2026-05-04 |
| **보고자** | jch1999 |
| **우선순위** | P1-High |
| **영역** | 싱글플레이 |
| **카테고리** | 기능 |
| **발생빈도** | 항상 |
| **상태** | 수정 완료 |

### 제목

기차 문(TrainDoor) 상호작용 키 입력 시 반응 없음

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 싱글플레이 |
| 플랫폼 | Windows 11 |

### 재현 절차

1. 튜토리얼 레벨 진입
2. 기차 문 앞으로 이동
3. 상호작용 범위 내에서 기차 문을 바라봄
4. 상호작용 키 입력

### 기대 결과

기차 문이 하이라이트(외곽선)되고, 상호작용 UI가 표시되며, 상호작용 키 입력 시 문이 열리거나 닫힌다.

### 실제 결과

기차 문에 가까이 가도 하이라이트가 표시되지 않는다. 상호작용 UI도 나타나지 않으며, 상호작용 키를 눌러도 아무 반응이 없다. 튜토리얼 지시사항(`Get on the train`)을 수행할 수 없는 상태이다.

### 원인 분석

**파일:** `Source/AO/Private/Train/AO_TrainDoor.cpp`  
**함수:** `AAO_TrainDoor::CanInteraction()` (53번째 줄)

```cpp
bool AAO_TrainDoor::CanInteraction(const FAO_InteractionQuery& InteractionQuery) const
{
    return false;   // 무조건 false 반환
}
```

상호작용 감지 태스크(`UAO_AbilityTask_WaitForInteractableTraceHit`)는 SphereTrace로 인터랙터블 액터를 감지한 뒤, `CanInteraction()`이 `true`를 반환하는 경우에만 하이라이트 및 어빌리티를 부여한다. `AAO_TrainDoor`는 이 함수를 무조건 `false`를 반환하도록 오버라이드하고 있어, Trace 히트 이후 `UpdateInteractionInfos()`의 필터링 단계에서 항상 제외된다.

부모 클래스 `AAO_BaseInteractable::CanInteraction()`은 `bInteractionEnabled` 플래그 및 비활성화 플레이어 목록을 올바르게 검사한 후 `true`를 반환하도록 구현되어 있다.

### 수정 내용

```cpp
// 수정 전
bool AAO_TrainDoor::CanInteraction(const FAO_InteractionQuery& InteractionQuery) const
{
    return false;
}

// 수정 후
bool AAO_TrainDoor::CanInteraction(const FAO_InteractionQuery& InteractionQuery) const
{
    return Super::CanInteraction(InteractionQuery);
}
```

### 검증 결과

수정 후 기차 문 상호작용 UI 정상 표시 및 문 개폐 동작 확인.

---


