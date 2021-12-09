# Critical Rendering Path

## Critical Rendering Path란?

HTML, CSS, JS가 파싱되어서 화면에 그려지는 과정을 의미한다.

## 순서

**Construction → Layout → Paint → Composition**

### 1.  Consturction 1 - DOM

- HTML → DOM토큰 → 노드 → DOM 트리
- 노드들은 토큰의 **위계서열을 기반으로 DOM 트리에 연결**된다.
- 중간에 `<script>` 태그를 만나면 블락킹되어 제어권을 넘겨준다.

💡 Preload Scanner DOM 트리를 생성하는 작업은 메인 스레드를 차지한다. 이때 preload scanner는 백그라운드에서 리소스를 받아오는 작업을 한다. 이미지나 async 또는 defer 속성을 가진 스크립트를 불러오는 작업을 동시에 진행할 수 있다.

### 2.  Construction 2 - CSSOM

브라우저는 CSS 규칙을 트리 형태로 변환한다.

- 점진적으로 증가하는 DOM과 달리 CSS는 그렇지 않다.
- 브라우저는 모든 CSS를 처리하고 수신할 때까지 페이지 렌더링을 막는다.
- CSS는 Cascading하게 **규칙을 덮어쓰기 때문에 완료되기 전까지 콘텐츠를 렌더링할 수 없다**.
- 토큰을 노드로 변환할 때 하위 노드가 **상위 스타일을 상속**한다.
- 규칙이 구체적일수록 DOM 트리 안에서 더 많은 노드를 지나야 하므로 더 많은 비용이 든다.
- CSSOM은 기본적으로 user agent의 스타일을 포함한다.
- 속도가 매우 빠름.

### 3.  Construction 3 - Render Tree

- DOM 트리와 CSSOM 트리가 결합된다.
- 보여지는 콘텐츠만 추가된다. 렌더트리에 포함되지 않는 경우는 다음과 같다.
  - `display: none;`인 경우
  - HTML의 `<head>`섹션

### 4.  Layout

Render Tree에서 어떤 노드들을 보여줄 지 결정했다면, 레이아웃 단계에서는 실제 화면에서 어느정도 위치에 놓이며, 사이즈는 어떻게될 지 등을 결정한다.

- 청사진을 그리는 단계. 이 정도 위치쯤에 배치가 되겠구나 가늠해보는 단계.
- 레이아웃단계에서는 화면의 크기를 고려해 사이즈를 결정한다.
- DOM노드의 수가 많을 수록 레이아웃은 더 길어진다.
- **렌더트리가 변경될 때**(노드 추가, 콘텐츠 변경, 박스 모델 스타일 변경 등) **레이아웃이 발생**한다.
- **레이아웃을 적게 하려면 일괄 업데이트를 하거나, 박스 속성의 애니메이션을 피해야한다**.
- (용어: Reflow는 파이어폭스 브라우저의 렌더링 엔진인 Gecko의 동작과정에 포함된다. 웹킷 렌더링 엔진에서는 Layout(배치)이라고 표기한다.)

### 5.  Paint

- 실제 화면에서 몇 픽셀을 차지할 지 등을 결정한다.
- 레이어가 같은 아이들끼리 묶는 과정을 진행해서 repainted될 때는 더 빠른 속도로 필요한 레이어만 변경하도록 한다.
- 처음에는 전체 화면이 그려지지만, 그 후에는 수정된 부분만 repainted된다. (브라우저가 최소의 영역만 repainting하는 데 최적화되어 있음.)
- 페인팅 과정은 매우 빠르다. (Render tree와 Layout에 비해서)
- CPU가 아닌 GPU로 실행하도록 하면 메인 스레드를 사용하지 않아도 되므로 성능향상에 도움이 된다.
- video 태그, canvas 태그나 opacity, transform, will-change 등의 CSS 속성들은 각자 자신의 레이어를 가진다.

  

### 6.  Composition

조립 단계. 만들어둔 레이어들을 정확한 위치나 단계(z-index를 이용)에 배치한다. 요소를 변경했을 경우 최고는 Composition 단계만 발생하는 것이다. 레이아웃 단계까지 가야한다면 이 동작을 꼭 해야하는지 다시 고민해보는 것이 좋다.



## 요약

- Critical rendering path는 브라우저에 HTML, CSS, js가 파싱되어 과정을 말한다.
- 순서는 Constructure(DOM 파싱, CSSOM파싱, 렌더트리 구축) -> Layout -> Paint -> Composition이다.
- 가능하면 Layout은 다시 발생하지 않는 것이 좋다.
- Paint는 레이어 단위로 이루어지며 , 이를 통해 빠른 속도로 필요한 부분만 교체할 수 있다.