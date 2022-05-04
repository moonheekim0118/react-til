리액트는 실제로 DOM 을 제어하는 방식이 아니라 중간의 가상 DOM, 즉 Virtual DOM 을 두어 개발의 편의성 (DOM 을 직접 제어하지 않는다)과 성능 (배치 처리로 DOM 변경) 을 개선했습니다. 

# 📍 Virtual DOM 이란

- 특정 기술이라기 보다는 아래와 같은 패턴에 가깝습니다.
- UI 의 가상 표현을 메모리에 저장 한 후 → React DOM 과 같은 라이브러리로 실제 DOM 과 동기화 합니다. *( 이러한 과정을 **재조정(Reconciliation)** 이라고 합니다. )*
- 즉 DOM 에 실제 변경사항을 바로 반영하는 것이 아니라, **Virtual DOM 에 변경 사항을 먼저 저장한 후, 이전과 비교 하여 실제로 변경이 일어난 부분만 실제 DOM 에 적용하는 것** 입니다.

<br/>
<br/>

# 📍 Virtual DOM 을 사용하는 이유

## 개발의 편의성

- 재조정에 의해 리액트의 *선언적 방식*을 가능하게 합니다. 리액트에게 원하는 UI 상태를 알려주면 DOM 이 그와 일치하도록 만듭니다.
- 따라서 어트리뷰트 조작, 이벤트 처리, 수동 DOM 업데이트와 같은 작업들이 추상화 됩니다.
- 왜 재조정이 선언적 방식을 가능하게 하는걸까요?
    - 선언적 방식이란 ? 필요한 것을 달성하는 과정을 하나 하나 기술하는 것(명령)이 아니라, 필요한 것이 어떤 것인지 기술하는 것입니다.
    - 재조정 과정이 없다면, DOM 의 변경사항들을 일일이 직접 명령해주어야 합니다. 우리는 V-DOM 을 이용한 재조정 과정을 통해서 이러한 ‘명령’을 할 필요가 없어졌습니다.

<br/>

## 성능

브라우저 렌더링 과정을 살펴보면 아래와 같습니다.

1. DOM tree 생성  : 렌더 엔진이 HTML 을 파싱하여 DOM 노드로 이루어진 트리를 생성한다.
2. Render tree 생성 : CSS 파일과 inline 스타일을 파싱하여 DOM+CSSOM = 렌더트리 생성
3. Layout (reflow) : 각 노드들의 스크린에서의 좌표에 따라 위치를 결정한다.
4. Paint (repaint) : 실제 화면에 그린다.

*DOM 요소에 변화가 발생 할 때마다 우리는 렌더트리를 다시 그림으로써 Render Tree 생성 → Reflow → Repaint 과정을 반복합니다.  VDom 은 이러한 성능의 문제를 해결해줍니다.* 

- 데이터 변경 시 전체 UI 는 일단 **Virtual DOM 에 먼저 반영** 됩니다. 이전 Virtual DOM 에 있던내용과 update 후의 내용을 **비교**하여 **바뀐 부분만 실제 DOM 에 적용**합니다. 아래 설명을 통해서 자세히 살펴봅시다.

- React 의 JSX 를 컴포넌트에서 Return 하면 바벨은 JSX 를 React.createElement() 호출로 컴파일 합니다. 실제 바벨에서 JSX 를 호출하면 JSX 가 객체 형태의 React Element 로 변환됩니다.
- React Element 는 DOM 요소의 가상 버전으로 , 가볍고(light), 상태를 가지지 않으며(stateless),불변성 (immutable) 을 유지합니다. 이러한 React Element 는 ReactDOM.render 에 의해서 실제 DOM 요소로 렌더됩니다.
- 하지만 React Element 는 불변성(immutable)을 가지기 때문에, 즉 변경이 불가능 하기 때문에 한번 요소를 만들었다면 데이터를 변경하기 위해서 그 자식이나 속성을 변경 할 수 없습니다.

```jsx
// before 
function SomeComponent(){
	return(
		<div>
				<div>우테코</div>
				<Poco/> 
		<div>
	)
}
```

```jsx
// after
function SomeComponent(){
	return(
		<div>
				<div>우테코</div>
				<Jun/> 
		<div>
	)
}
```

- 위 예시에서 SomeComponent 의 바뀐 부분은 Poco → Jun 밖에 없는데, React Element 로 만들어진 SomeComponent 의 자식속성만 변경 할 수 없으니 다시 전체 컴포넌트의 React Element를 새로 생성하여 ReactDOM.render 로 전송하는 수밖에 없습니다.
- 이러한 문제점을 해결하기 위해 리액트는 Virtual DOM 을 이용하는 것입니다. 모든 React DOM object 는 그에 대응하는 Virtual DOM object 가 있습니다. 그리고 V-DOM object 는 각각의 React DOM object 에 맵핑됩니다.
- 데이터가 업데이트 되면, 바뀐 데이터를 기반으로 React.createElement 를 통해 JSX Element 를 렌더링 하고, 이 때 모든 Virtual DOM object 가 업데이트 됩니다. 이 때 Virtual DOM 이 업데이트 되면, 리액트는 업데이트 이전의 Virtual DOM 의 스냅샷과 비교하여 정확히 어떤 Virtual DOM 이 바뀌었는지 추적합니다. (Diffing 알고리즘)
    - *Element 의 속성만 변경 된 경우 ? 속성 값만 업데이트*
    - *Element 의 태그 혹은 컴포넌트가 변경된 경우 ? 해당 노드를 포함한 하위 모든 노드를 언마운트 후, 새로운 Virtual DOM 으로 대체*
- **이 후 변경된 부분만 실제 DOM 에 적용하게 됩니다.**

<br/>
<br/>

# 📍 Virtual DOM이 무조건 빠를까?

- 답은 NO
- 정보 제공만하는 어플리케이션 (ex 위키피디아) 이라면 아무런 인터렉션이 일어나지 않기 때문에, DOM 트리의 변화가 일어나지 않아 일반 DOM 의 성능이 더 좋을 수도 있습니다.

<br/>
<br/>

# 🧐 생각해보기

## 우리는 JavaScript만으로 앱을 구성할 때 명령형으로 작성했을까요? 선언형으로 작성했을까요?

- 정답부터 말하자면 명령형이요! DOM 의 변경사항을 일일이 코드로 직접 반영해줘야 했기 때문에 명령형입니다.
- 예를들어 isError 란 상태가 true일 때 html 엘리먼트 내부의 text 를 바꾸어주는 작업을 한다고 했을 때에도 우리는 직접 해당 엘리먼트를 선택하여 textContent 를 수정해줘야 했습니다.
- 반면 리액트로 코드를 작성할 경우, DOM 을 직접적으로 조작할 필요가 없습니다.

## React 이전의 SPA 라이브러리들은 무엇이 있었을까요?

- Jquery, Backbone, AngularJS 등 많은 라이브러리가 있었고, 모두 MV* 패턴을 따라갔습니다.
- 즉, Model (데이터) 가 변경되면 이를 구독하고 있는 View가 업데이트 됩니다.

## React 탄생 배경은 어떻게 될까요?

웹사이트가 점점 복잡해짐에 따라서 사용자의 인터렉션에 따라서 DOM 을 업데이트 해줘야할 일이 늘었고 이에 따라 페이스북팀은 문제점들을 발견하고 해결했습니다.

리액트와 다른 프레임워크의 가장 큰 차이점은 바로 데이터바인딩입니다. 리액트 이전의 프레임워크는 양방향 바인딩 (MV* 패턴) 을 채택했고 리엑트는 단방향 바인딩 (Flux 패턴) 을 채택했습니다.  이를 페이스북팀이 겪은 문제와 함께 설명해봅니다.

### 양방향 바인딩의 문제점

- 리액트 이전의 프레임워크들은 MV* 모델 즉, MODEL - VIEW 를 분리하는 방식으로 이러한 문제들을 해결했습니다. MV* 모델에서 Model 은 데이터를 가지고 있고, Model 의 데이터가 변경되면 View 가 업데이트 됩니다. 즉 , 양방향 데이터 바인딩 (모델과 뷰가 서로 묶여 있으므로)이라고 합니다.
- 페이스북 알림 버그
    - 페이스북에 로그인 했을 때 상단의 메시지 아이콘에 알림 1 이 떠 있지만, 그 메시지 아이콘을 클릭해서 들어가보면 아무런 메시지가 없는 버그가 있었습니다. 알림은 사라지겠지만, 몇 분 뒤 알림이 다시 나타나고 여전히 아무런 메시지도 나타나지 않는 문제도 발생했습니다.
    - 이러한 문제는 복잡한 구조에서 모델과 뷰에서 이에 대한 처리를 각각 진행하다 보니 사이클이 꼬여버리는게 원인이었다고 합니다.
    
    ![https://bestalign.github.io/static/24c9f4a0d26c1ee8c60ba9485771206c/0a47e/03.png](https://bestalign.github.io/static/24c9f4a0d26c1ee8c60ba9485771206c/0a47e/03.png)
    
    - 위와 같이 데이터를 가지고 있는 모델이 렌더링을 위해 뷰로 데이터를 보내는데, 사용자와의 상호작용이 복잡해지다 보니, 사용자와의 상호작용이 일어난 뷰에서, 모델로 데이터를 업데이트 해줘야 하는 경우가 생긴겁니다.
    - 그리고 의존성에 의해서 모델이 또 다른 모델을 업데이트 해줘야 하는 경우도 생기구요.
    
    ![https://bestalign.github.io/static/9057e46a6886b24fe34439f7b5c78f81/0a47e/04.png](https://bestalign.github.io/static/9057e46a6886b24fe34439f7b5c78f81/0a47e/04.png)
    
    - 또한 이러한 업데이트들은 비동기적으로 동작할 수도 있고, 하나의 업데이트가 여러 업데이트들을 트리거 할 수도 있습니다.
    - 즉 위와 같이 아주 복잡한 데이터의 흐름구조가 생기면서 데이터 흐름을 디버깅하기 매우 어려워졌습니다.
    

이러한 문제점을 해결하기 위해 페이스북팀은 단방향 데이터 흐름을 도입합니다. 

### 단방향 데이터 흐름

페이스북 개발자들의 초기 아이디어는 이랬습니다.

- *데이터 변경이 일어날 때마다 View 를 수정하는게 아니라,  데이터 변경이 일어날 때마다 현재의 View 를 없애버리고 (!) 새로운 View 를 넣자!*
- 하지만 이렇게 될 경우 좋지않은 유저 경험과 성능상 문제가 일어날 수 있겠죠? 이러한 아이디어를 채택하되, 아이디어의 문제점을 해결하기 위해 리액트가 등장하게 됩니다.
- 가장 큰 개선점은 Update 대신, **재조정(Reconcililation)** 을 한다는 것입니다. 위에서 설명한 Virtual DOM 을 사용해서 리액트는 데이터가 수정될 때 마다 View 를 통째로 갈아끼우는 대신 재조정을 하게 됩니다.
- 그리고 리액트는 단방향 흐름을 제어하기 위한 flux 패턴을 도입합니다. flux 패턴에 대해서는  [https://bestalign.github.io/translation/cartoon-guide-to-flux/](https://bestalign.github.io/translation/cartoon-guide-to-flux/) 를 참고하는 것을 추천드립니다.

![https://bestalign.github.io/static/e6fc90e56d6c9471933c75d91ad9f35a/1cfc2/05.png](https://bestalign.github.io/static/e6fc90e56d6c9471933c75d91ad9f35a/1cfc2/05.png)

결론적으로 리액트는 웹에서 사용자와의 인터렉션이 증가하여 복잡도가 올라감에 따라, 기존의 양방향 데이터 바인딩에서 오는 여러 문제점들을 해결하기 위해 등장했습니다. 이를 단방향 데이터바인딩으로 해결하고자 했고, 단방향 데이터 바인딩 시의 문제점 (뷰를 계속해서 새로 그려줘야한다는 것)을 버츄얼 돔으로 해결했습니다.

<br/>

# 참고 자료

- https://ko.reactjs.org/docs/faq-internals.html#gatsby-focus-wrapper
- https://www.youtube.com/watch?v=PN_WmsgbQCo
- https://d2.naver.com/helloworld/9297403
- https://youtu.be/XxVg_s8xAms
- https://bestalign.github.io/translation/cartoon-guide-to-flux/
