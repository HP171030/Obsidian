# 개요

- 3세대 UI 시스템이라 보면 된다
- IMGUI - UGUI - UIToolkit 의 형식으로 내려오고 있는 유니티에서 제공하는 런타임 UI시스템 및 에디터 확장 기능이다
- 웹 로직을 따온 UI시스템이라 객체도 비슷한 모양으로 사용하고 있다 
- HTML or XML=> UXML (구조 및 레이아웃)
- CSS => USS (스타일)
- JavaScript => C# (기능 및 상호작용)
---
# VisualElement
- UI Toolkit에서 사용하는 기본 구성 단위이다.
- 트리 구조로 구성되어 HTML의 DOM 요소와 유사하게 동작한다.

# UI Document
- UI를 어떻게 구성할지 리소스를 넣는 세팅 컨트롤러와 같은 기능을 한다
- **Panel Settings** : UGUI에서 사용하던 캔버스와 비슷한 기능을 한다. 렌더링 방식과 입력이벤트 처리방식을 설정한다

---
# Layout


# 1. margin ,border ,padding
![[화면 캡처 2025-07-16 093338.png]]

- 요소 테두리에 여백을 주는 구성 계층이다
- 바깥부터 margin ,border ,padding순으로 여백을 줄 수 있다.

# 2. Align

![[화면 캡처 2025-07-16 093701.png]]

부모의 flex Direction 상태에 따라 정렬 방식을 정할 수 있다
# - Align items : 
자식 요소들을 교차축 방향으로 어떻게 정렬할지 설정하는 속성이다.

- Flex start : 축 시작부에 정렬
- Flex Center : 축 중앙에 정렬
- Flex end : 축 끝 부분에 정렬
- Stretch : 자식 요소의 크기를 교차축 전체에 맞춰 늘림
# - Justify Content : 
자식 요소들을 주축 방향으로 어떻게 배치할지 정의하는 속성이다. 요소 간의 간격, 시작점 위치 등을 지정할 수 있다.

- Flex start : 주축의 **시작 지점**부터 요소들을 나열
- Center : 주축의 **중앙 지점**부터 요소들을 나열
- Flex end : 주축의 **끝 지점**부터 요소들을 나열
- Space between : 양 끝 고정하고 요소 사이에 space 추가
- Space around 요소 상하좌우에 space 추가


# USS

UXML 내 구성요소들을 스타일링할 때, UI 규모가 커지거나 여러 요소 간의 시각적 일관성을 유지해야 할 경우, USS를 활용하면 스타일을 재사용하고 관리할 수 있어 생산성과 유지보수성이 향상된다.

# Pseudo class 

UI 요소에 사용자의 상호작용 상태를 반영할 수 있는 클래스를 **Pseudo class**라고 하며, 대표적인 종류는 다음과 같다.


- `:hover` – 마우스가 올라왔을 때
	
- `:active` – 클릭 중일 때
	
- `:focus` – 포커스가 맞춰졌을 때
	
- `:checked` – 체크된 상태일 때 (토글, 토글 버튼 등)
	
- `:disabled` – 비활성 상태일 때

# Position 

- 레이아웃의 영향을 받음 **Relative** 아니라면 **Absolute**


Flex

- 화면이 부족해 요소가 줄어드는 정도 **Shrink**
- 화면이 남아 요소가 그 남은 공간을 채우는 정도 **Grow**

---

# Scripting

```Csharp
//스크립팅 할 수 있도록 요소를 참조해올 수 있게하는 함수
var panelSheet = root.Q<VisualElement>("elementName")

//스타일 클래스를 비쥬얼엘리먼트에 추가
panelSheet.AddToClassList("styleClassName");

//스타일 클래스를 비쥬얼엘리먼트에서 제거
panelSheet.RemoveFromClassList("styleClassName");

//비쥬얼엘리먼트 내에 해당 스타일클래스가 있는지 확인
panelSheet.ClassListContains("styleClassName");

//스타일 리스트를 끄거나 켜는 방식으로 호출
//ex) 켜져있으면 끄고 꺼져있으면 키는방식
panelSheet.ToggleInClassList("styleClassName");
```
---
# Events

|이벤트 이름|설명|
|---|---|
|`TransitionRunEvent`|트랜지션 애니메이션이 **시작되기 전**에 발생|
|`TransitionStartEvent`|트랜지션 애니메이션이 **실제로 시작**될 때 발생|
|`TransitionEndEvent`|트랜지션이 **정상적으로 끝났을 때** 발생|
|`TransitionCancelEvent`|트랜지션이 **중단되었을 때** 발생|

| 이벤트 이름            | 설명                                    |
| ----------------- | ------------------------------------- |
| `MouseEnterEvent` | 마우스가 해당 요소에 **진입**할 때                 |
| `MouseLeaveEvent` | 마우스가 해당 요소에서 **벗어날 때**                |
| `MouseDownEvent`  | 마우스 버튼을 **눌렀을 때**                     |
| `MouseUpEvent`    | 마우스 버튼을 **뗐을 때**                      |
| `ClickEvent`      | **클릭이 완료되었을 때** (Down → Up 모두 일어난 경우) |
| `MouseMoveEvent`  | 마우스가 **이동 중일 때**                      |
| `WheelEvent`      | 마우스 휠을 **스크롤**할 때                     |

|이벤트 이름|설명|
|---|---|
|`FocusInEvent`|포커스를 얻었을 때|
|`FocusOutEvent`|포커스를 잃었을 때|
|`KeyDownEvent`|키를 눌렀을 때|
|`KeyUpEvent`|키를 뗐을 때|
|`NavigationSubmitEvent`|엔터/컨펌 입력이 들어왔을 때 (게임패드 지원)|


```csharp
//ex) 콜백을 붙이면 된다
element.RegisterCallback<MouseDownEvent>(OnMouseDown);
```


---


## ETC

- 씬의 첫번째 프레임에는 그 이전상태가 존재하지 않기 때문에 트랜지션 애니메이션은 첫번째 프레임은 건너뛰고 호출해야한다. (모듈화 가능성제시)