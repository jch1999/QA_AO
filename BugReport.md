# Bug Report — AVaOut: Avatar Out

---

## BUG-001

| 항목 | 내용 |
|---|---|
| **관련 체크리스트** | M-001 |
| **보고일** | 2026-05-03 |
| **보고자** | jch1999 |
| **우선순위** | P1-High |
| **상태** | 수정 완료 |

---

### 제목

메인 메뉴에서 CreateGame 버튼 클릭 시 방 생성 다이얼로그가 출력되지 않음

---

### 환경

| 항목 | 내용 |
|---|---|
| 엔진 버전 | Unreal Engine 5.6.1 |
| 실행 환경 | 에디터 PIE (Play In Editor) |
| 네트워크 구성 | 호스트 단독 실행 |
| 플랫폼 | Windows 11 |

---

### 재현 절차

1. 에디터에서 `LV_MainMenu` 레벨을 PIE로 실행
2. 메인 메뉴 화면에서 **Create Game** 버튼 클릭

---

### 기대 결과

방 이름 입력 등 룸 생성 정보를 입력하는 `UAO_HostDialogWidget` 다이얼로그가 화면에 출력된다.

---

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

---

### 원인 분석

**파일:** `Source/AO/Private/UI/Widget/AO_MainMenuWidget.cpp`  
**함수:** `UAO_MainMenuWidget::OnClicked_Host()` (16번째 줄)

`UAO_HostDialogWidget`을 생성하고 `AddToViewport()` 한 직후 `SetVisibility(ESlateVisibility::Hidden)`을 호출하고 있었다. 위젯은 정상 생성되어 뷰포트에 추가되지만 즉시 숨김 처리되어 출력되지 않았다.

Widget Reflector로 추적 시 해당 위젯의 Visibility가 `Hidden` 상태인 것으로 확인되었다.

---

### 수정 내용

**파일:** `Source/AO/Private/UI/Widget/AO_MainMenuWidget.cpp`, 29번째 줄

```cpp
// 수정 전
Dialog->SetVisibility(ESlateVisibility::Hidden);

// 수정 후
Dialog->SetVisibility(ESlateVisibility::Visible);
```

---

### 검증 결과

수정 후 컴파일 및 PIE 실행 시 Create Game 버튼 클릭에 정상적으로 방 생성 다이얼로그가 출력되는 것을 확인.

---
