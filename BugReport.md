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
| **상태** | 미수정 |

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

PlayerInput에 대한 키 바인딩이 제대로 되지 않았을 것으로 추정. 추가 분석 필요.

### 수정 내용

미수정

### 검증 결과

해당 없음

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
| **상태** | 미수정 |

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

`GA_Sprint`에 대한 분석 필요. 달리기가 기본 이동 속도로 설정되어 있을 가능성 추정.

### 수정 내용

미수정

### 검증 결과

해당 없음

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
| **상태** | 미수정 |

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

분석 필요

### 수정 내용

미수정

### 검증 결과

해당 없음

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
| **상태** | 미수정 |

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

분석 필요

### 수정 내용

미수정

### 검증 결과

해당 없음

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
| **상태** | 미수정 |

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

`PickupItem()` 129번째 줄에서 null 포인터 역참조 발생 (`EXCEPTION_ACCESS_VIOLATION reading address 0x1f0`). 인벤토리 슬롯이 가득 찬 상태에서 빈 슬롯을 찾지 못했음에도 슬롯 포인터를 null 체크 없이 접근한 것으로 추정.

### 수정 내용

미수정

### 검증 결과

해당 없음

---
