# Cinematic Transition Design

## Goal

현재 원페이지는 장면의 분위기와 개별 섹션의 정체성은 충분히 좋다.  
이번 보강의 목표는 전체를 재설계하는 것이 아니라, 섹션과 섹션 사이의 연결 전환을 강화해 `장면 모음`보다 `하나의 통과 경험`처럼 느껴지게 만드는 것이다.

핵심 원칙은 다음과 같다.

- 전환 강도는 `절제된 시네마틱`으로 유지한다.
- 기존의 검은 여백, 침묵감, 느린 호흡은 유지한다.
- 새 인터랙션은 섹션 내부 전체보다 `섹션 경계의 20% 구간`에 집중한다.
- 장면은 교체되는 것이 아니라, 이전 장면의 요소가 다음 장면으로 `변형`되어야 한다.

## Scope

이번 작업의 범위는 아래 네 가지다.

1. 섹션 간 morph transition 추가
2. Echo / Reflection / Disappearance의 커서 반응 보강
3. 장면 레이어, 광점, 파티클의 소프트 패럴랙스 추가
4. HUD와 좌측 텍스트의 전환 타이밍 미세 조정

이번 작업에서 하지 않는 것:

- 전체 레이아웃 구조 변경
- HUD 재디자인
- 섹션 텍스트 카피 재작성
- Three.js 등 대형 렌더러 도입
- 각 영상/이미지 에셋 자체 수정

## Desired Experience

사용자는 스크롤하면서 장면이 끊겨 바뀐다고 느끼지 않아야 한다.  
대신 작은 점, 반복 점열, 잔상, 반사축, 고립된 광점, 분해된 파편이 하나의 생명체처럼 천천히 형태를 바꾸며 이어져야 한다.

체감상 변화는 분명하되 과장되면 안 된다.

- 정적인 구간은 여전히 정적으로 느껴져야 한다.
- 전환 구간에서만 다음 장면의 성질이 스며들어야 한다.
- 마우스 반응은 분위기를 해치지 않는 수준으로 얕아야 한다.

## Transition System

전환은 각 섹션 progress의 마지막 약 20%와 다음 섹션 progress의 첫 20%를 함께 사용해 만든다.

각 경계는 아래처럼 정의한다.

### Silence -> Loop

- 현재의 중심 점이 좌우로 번지며 복제의 씨앗이 된다.
- 단일 광점의 glow spread가 늘어나고, 좌우 방향으로 얇은 반복 점이 나타나기 시작한다.
- 영상 레이어는 그대로 유지하되 canvas overlay에서 점 복제 구조가 생겨난다.

### Loop -> Echo

- 반복 점열이 또렷한 구조를 잃고 잔상으로 무너진다.
- 점의 개수는 유지하되 opacity와 blur가 증가하며 motion trail처럼 보이게 바뀐다.
- 스크롤이 깊어질수록 점열의 질서보다 흐려진 흔적이 우세해진다.

### Echo -> Reflection

- 흩어진 잔상이 중앙의 수평축으로 정렬되기 시작한다.
- trail 입자 일부가 위아래로 짝을 이루며 reflection 구조를 만든다.
- 지연 복제 커서 반응이 이 구간부터 자연스럽게 살아난다.

### Reflection -> Isolation

- 복제 레이어와 반사 라인이 점차 사라지고, 가장 중심에 가까운 광점만 남는다.
- reflection의 대칭성은 유지되지 않고 조용히 해체된다.
- 안개와 glow만 남기고 입자 밀도는 줄인다.

### Isolation -> Disappearance

- 고립된 중심 점이 미세 입자로 부서지기 시작한다.
- particle count는 잠깐 증가하지만 전체 밝기는 올라가지 않도록 opacity를 낮춘다.
- 마지막에는 점의 중심성이 사라지고, 파편의 잔존감만 남는다.

## Interaction Design

### Silence

- 현재보다 아주 약한 광점 반응만 유지한다.
- 마우스 이동에 따라 glow가 미세하게 따라가되, 위치 이동보다 크기 변화 위주로 반응한다.

### Loop

- 영상 자체는 흔들리지 않는다.
- 구조감은 overlay layer에서만 느껴지게 한다.
- 마우스 반응은 거의 제거하고, 스크롤 기반 깊이 변화만 사용한다.

### Echo

- 커서 잔상이 2초 안팎 남아야 한다.
- trail point의 radius와 opacity 감쇠를 늦춰 여운이 더 길게 머물게 한다.
- 오래된 잔상은 더 흐리고 크게 보여야 한다.

### Reflection

- 현재 커서 외에 0.3초, 0.7초 지연 위치를 사용한다.
- 세 레이어는 완전 일치하지 않고 미세하게 어긋나야 한다.
- 수평 반사축 근처에서 지연 복제 반응이 가장 잘 보이도록 한다.

### Isolation

- 반응을 거의 제거한다.
- 커서 이동은 사실상 느껴지지 않아야 한다.

### Disappearance

- 커서 주변 파편이 아주 미세하게 퍼진다.
- 가까운 입자만 약간 밀려나고, 전체 장면이 요동치면 안 된다.

## Soft Parallax

패럴랙스는 깊이감을 보조하는 용도로만 사용한다.

- background media layer: 거의 고정, 아주 느린 scale 변화
- glow / fog layer: 약한 translate
- particle / trail layer: 가장 많이 반응
- text / HUD: 거의 고정, 필요할 때만 opacity 조절

추천 수치는 다음과 같다.

- background translate: 0px ~ 6px
- glow translate: 4px ~ 12px
- particle translate: 10px ~ 20px
- scale shift: 1.00 -> 1.06
- blur shift: 0 -> 4px

중요한 점은, 영상 레이어 자체가 흔들리는 느낌보다 overlay depth가 살아나는 느낌이어야 한다는 것이다.

## Technical Approach

기존 `index.html` 단일 파일 구조는 유지한다.

구현은 세 부분으로 나눈다.

### 1. Transition Progress Model

- 현재 active section progress 외에 `boundary progress` 개념을 추가한다.
- 현재 섹션의 끝 구간과 다음 섹션의 시작 구간을 공통 파라미터로 환산한다.
- 이 값을 이용해 morph overlay를 렌더링한다.

### 2. Overlay Renderer Expansion

- 현재 canvas overlay에 섹션별 전환 전용 렌더 함수를 추가한다.
- 기본 섹션 렌더와 전환 렌더를 분리한다.
- 장면 media는 유지하고, morph는 canvas에서 수행한다.

예상 함수 구조:

- `renderSceneOverlay(progress, key)`
- `renderTransitionOverlay(fromKey, toKey, boundaryProgress)`
- `getBoundaryState()`

### 3. Interaction Buffers

- trail, delayedTrail, delayedTrailFar를 섹션별로 다르게 사용한다.
- Echo / Reflection에서만 더 적극적으로 사용하고, Isolation에서는 비운다.
- Disappearance에서는 별도 파편 drift 계산을 추가한다.

## Error Handling And Safeguards

- 사용자가 빠르게 스크롤해도 전환 로직이 깨지지 않아야 한다.
- active section 계산이 튀더라도 boundary progress는 0~1로 clamp한다.
- 영상 레이어는 항상 기본 장면 역할을 유지하고, overlay 실패 시에도 화면이 비어 보이면 안 된다.
- reduced motion 환경에서는 전환 강도를 낮추거나 생략한다.

## Testing

최소 확인 항목은 다음과 같다.

1. Intro -> Silence 자동 전환이 깨지지 않는지
2. Silence -> Loop에서 점이 복제되는 감각이 생기는지
3. Loop 영상은 고정되고 overlay만 깊이를 만드는지
4. Echo에서 커서 잔상이 이전보다 길게 남는지
5. Reflection에서 지연 복제가 눈에 띄되 과하지 않은지
6. Isolation은 여전히 가장 정적인 장면인지
7. Disappearance에서 파편화가 조용하게 보이는지
8. 모바일에서 텍스트/HUD가 화면을 과도하게 가리지 않는지

## Recommendation

구현 순서는 아래가 가장 안정적이다.

1. boundary progress 계산 추가
2. 섹션 간 transition overlay 추가
3. Echo / Reflection / Disappearance 인터랙션 보강
4. 소프트 패럴랙스 미세 조정
5. HUD / 텍스트 타이밍 정리

이번 작업의 성공 기준은 “더 화려해졌다”가 아니라,  
`장면들이 하나의 감각으로 이어진다`는 인상이 분명해지는 것이다.
