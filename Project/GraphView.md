# Unity GraphView 개요

노드 기반 GUI 에디터를 제작할 수 있는 UIElements 기반 툴킷
직관적인 인터페이스로 비즈니스 로직, 대화 시나리오, 상태 머신 등을 구성하는 데 용이함

---

## 핵심 구조 요약

GraphView
├── NodeA
│ └── PortA (Output)
├── NodeB
│ └── PortB (Input)
└── Edge (PortA → PortB)


#### 대표 속성

|속성/메서드|설명|
|---|---|
|`AddElement(GraphElement)`|요소 추가 (Node, Edge 등)|
|`RemoveElement(GraphElement)`|요소 제거|
|`graphViewChanged`|요소 변경 이벤트 훅|
|`nodes`, `edges`|그래프 내 노드/엣지 컬렉션 접근|

---

###  Node (GraphElement 상속)

- 하나의 **노드 블록**을 나타냄
    
- 내부에 포트(Port)들을 가지고 있음
    
- 확장 UI 가능 (`mainContainer`, `extensionContainer`)
    

|속성|설명|
|---|---|
|`title`|노드 타이틀 텍스트|
|`inputContainer` / `outputContainer`|포트를 담는 컨테이너|
|`mainContainer`|노드의 메인 영역|
|`extensionContainer`|확장 가능한 내용 영역|
|`capabilities`|이동, 삭제 등 허용 여부|

#### 위치 설정

`node.SetPosition(new Rect(x, y, width, height));`

---

### Port (GraphElement 상속)

- **노드 간 연결을 위한 입출력 단자**
    
- `Port.Create()`로 생성 가능
    
- Direction 설정 필요: `Direction.Input` 또는 `Direction.Output`
    

#### 주요 속성

|속성|설명|
|---|---|
|`direction`|입력 또는 출력 방향|
|`portType`|연결 타입 지정 (e.g., int, float)|
|`portName`|포트의 이름|
|`connections`|연결된 엣지 목록|
|`Connect(edge)` / `Disconnect(edge)`|포트 연결 관리|

---

###  Edge (GraphElement 상속)

- **두 포트를 연결하는 선**
    
- 비주얼 상으로는 Bezier 커브로 표시됨
    
- 하나의 Edge는 **하나의 Output → 하나의 Input**만 연결 가능
    

#### 주요 속성

|속성|설명|
|---|---|
|`input` / `output`|연결된 포트 객체|
|`inputNode` / `outputNode`|포트가 속한 노드|
|`Reconnect()`|연결 재설정|
|`Disconnect()`|연결 해제|

---

##  GraphElement (공통 기반 클래스)

- `Node`, `Port`, `Edge` 등 모든 요소의 상위 클래스
    
- 선택, 이동, 스타일 등 공통 속성 포함
    

#### 공통 속성

|속성|설명|
|---|---|
|`capabilities`|요소 기능 설정 (이동, 삭제, 복사 등)|
|`viewDataKey`|직렬화 키|
|`style`, `AddToClassList()`|스타일 설정|

---

## 기타 요소

|구성요소|설명|
|---|---|
|`Blackboard`|변수 목록 및 외부 데이터 관리|
|`MiniMap`|그래프 전체를 축소해서 보여주는 미니맵|
|`Toolbar`|상단 UI 버튼/검색창 등|

---

## 직렬화 & 버그 이슈 예시

###  Json 직렬화/역직렬화 흐름

- 각 노드 및 포트의 고유 ID 저장
    
- `Edge`는 연결된 포트의 ID로 구성됨
    
- 불필요한 ID가 남아 있으면 연결 오류 발생 가능
    

### 대표적인 버그

- 삭제된 노드 ID가 JSON에 남아 **존재하지 않는 노드와 연결**됨
    
- 해결 방법: 불러오기 전 유효성 검사 및 ID 정리 필요
    

---

## 참고 구현 예시

`var node = new Node { title = "MyNode" }; var portOut = Port.Create(Direction.Output, typeof(float)); var portIn = Port.Create(Direction.Input, typeof(float));  node.outputContainer.Add(portOut); anotherNode.inputContainer.Add(portIn);  var edge = portOut.ConnectTo(portIn); graphView.AddElement(edge);`

---

## 요약

|구성 요소|설명|
|---|---|
|GraphView|전체 그래프 시스템의 컨테이너|
|Node|하나의 블록 단위, 포트를 포함|
|Port|입출력 단자|
|Edge|포트 간의 연결선|
|GraphElement|공통 부모 클래스 (선택, 이동 등 기능 포함)|
