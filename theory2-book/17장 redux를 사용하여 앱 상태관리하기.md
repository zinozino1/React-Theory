```js

* 보통 스토어의 내장 함수은 dispatch, subscribe 함수를 사용하지 않고 react-redux 라이브러리에서 제공하는 유틸함수 (connect, provider)를 사용하여 리덕스 관련 작업을 처리한다.


*** 리액트 프로젝트에서 리덕스를 사용할 때 가장 많이 사용하는 패턴은 프레젠테이셔널 컴포넌트와 컨테이너 컴포넌트를 분리하는 것이다.

- 프레젠테이셔널 컴포넌트 : 상태 관리가 이루어지지 않고 그저 props만 받아 와서 화면 UI 를 보여주기만 하는 컴포넌트

- 컨테이너 컴포넌트 : 리덕스와 연동되어 있는 컴포넌트로, 리덕스로부터 상태를 받아 오기도 하고 리덕스 스토어에 액션을 디스패치하기도 한다.


*** 리덕스 코드 디렉토리 구조

1) 일반적 구조

- actions(액션 생성 함수 정의)
- constants(액션 타입 정의)
- reducers(리듀서 정의)

-> 코드르 종류에 다라 다른 파일에 작성하여 정리할 수 잇어서 편리하지만 새로운 액션을 만들 때마다 세 종류의 파일을 모두 수정해야 하기 때문에 불편함. 하지만 리덕스 공식 문서에서 이 구조를 택하고 있음.

2) Ducks 패턴

- modules(액션 타입, 액션 생성 함수, 리듀서 함수를 기능별로 파일 하나에 몰아서 다 작성하는 방식)
- 이번 프로젝트에서는 Ducks패턴을 사용하여 코드 작성.
- Ducks 패턴을 사용하여 액션 타입, 액션 생성 함수, 리듀서를 작성한 코드를 모듈이라고 한다.



* 모듈 작성하기

1) 액션 타입 정의하기

const INCREASE = "counter/INCREASE";
const DECREASE = "counter/DECREASE";
-> 문자열 안의 내용은 모듈이름/액션이름과 같은 형태로 작성해야 나중에 프로젝트가 커졌을 때 액션의 이름이 충돌되지 않게 해준다.

2) 액션 생성 함수 만들기

export const increase = () => {
    return {
        type: INCREASE,
    };
};
export const decrease = () => {
    return {
        type: DECREASE,
    };
};

-> export 꼭 써줘야 함

3) 초기 상태 및 리듀서 함수 만들기

- 액션 생성함수는 export, 리듀서는 export default로 내보내준다.

// 초기 상태 정의

const initialState = {
    number: 0,
};

// 리듀서 함수 정의

const counter = (state = initialState, action) => {
    switch (action.type) {
        case INCREASE:
            return {
                number: state.number + 1,
            };
        case DECREASE:
            return {
                number: state.number - 1,
            };
        default:
            return state;
    }
};

4) 루트 리듀서 만들기

- 나중에 createStore 함수를 사용하여 스토어를 만들 대는 리듀서를 하나만 사용해야한다.
- 그렇기 때문에 기존에 만들었던 리듀서를 하나로 합쳐주어야 하는데, 이 작업은 redux에서 제공하는 combineReducers라는 유틸 함수를 사용하여 쉽게 처리할 수 있다.
- modules 디렉토리에 index.js 라는 파일로 만들기.

import { combineReducers } from "redux";
import counter from "./counter";
import todos from "./todos";

const rootReducer = combineReducers({
    counter,
    todos,
});

export default rootReducer;

-> 파일 이름을 index.js 로 설정해주면 나중에 불러올 때 디렉터리 이름까지만 입력하여 불러올 수 있다.
- import rootReducer from "./modules"




*** 리액트 애플리케이션에 리덕스 적용하기

- 스토어를 만들고 리액트 애플리케이션에 리덕스를 적용하는 작업은 src 디렉터리의 index.js에서 이루어진다.

1) 스토어 만들기

import { createStore } from "redux";
import rootReducer from "./modules";

const store = createStore(rootReducer);

2) Provider 컴포넌트를 사용하여 프로젝트에 리덕스 적용하기

- 리액트 컴포넌트에서 스토어를 사용할 수 있도록 App 컴포넌트를 react-redux에서 제공하는 Provider 컴포넌트로 감싸준다. 이 컴포넌트를 사용할 때는 store를 props로 전달해줘야 함.

3) redux devtools의 설치 및 적용

- redux 개발자 도구이며 크롬 확장 프로그램으로 설치하여 사용할 수 있음

import { composeWithDevTools } from "redux-devtools-extension";

const store = createStore(rootReducer, composeWithDevTools());

4) 컨테이너 컴포넌트 만들기

- 리덕스 스토어와 연동된 컴포넌트를 컨테이너 컴포넌트라고 함
- src/containers 에 컨테이너 컴포넌트 생성
- 이 컴포넌트를 리덕스와 연동하려면 react-redux에서 제공하는 connect 함수를 사용해야 함
- connect(mapStateToProps, mapDispatchToProps);
- mapStateToProps는 리덕스 스토어 안의 상태를 컨테이너 컴포넌트의 props로 넘겨주기 위해 설정하는 함수
- mapDispatchToProps는 리덕스 스토어 안의 액션 생성함수를 컨테이너 컴포넌트의 props로 넘겨주기 위해 설정하는 함수

- connect함수를 호출하고 나면 또 다른 함수를 반환한다. 반환된 함수에 컴포넌트를 파라미터로 넣어주면 리덕스와 연동된 컴포넌트가 만들어진다.

-> const makeContainer = connect(mapStateToProps, mapDispatchToProps);
makeContainer(타겟 컴포넌트);


* redux store(리액트에 종속되어 있지 않은 라이브러리)의 내장 함수은 dispatch, subscribe 함수를 직접적으로 사용하지 않고 react-redux의 connect, provider를 이용한다.


***** 리덕스 도입할 때 중요한 3가지 작업

1 - store 생성 및 모듈 정의(액션타입, 액션생성함수, 초기 상태값, 리듀서)
2 - container 컴포넌트 작성 & store와 connect
3 - presentational component에서 logic 작성


* 리덕스를 더 편하게 사용하기

1) 액션 생성 함수, 리듀서를 작성할 때 redux-actions라는 라이브러리 사용

- 액션 생성 함수를 더 짧은 코드로 작성 가능
- 리듀서를 작성할때도 switch/case가 아닌 handleActions라는 함수를 사용하여 각 액션마다 업데이트 함수를 설정하는 형식으로 작성할 수 있음

** 파라미터가 필요할 경우

액션에 필요한 추가 데이터는 payload라는 이름을 사용한다.

ex1)
const MY_ACTION = "sample/MY_ACTION"
const myAction = createAction("MY_ACTION");
const action = myAction('hello');

action => {
    type : My_ACTION,
    payload : 'hello',
}

ex2)
-> 액션 생성 함수에서 받아 온 파라미터를 그대로 payload에 넣는 것이 아니라 변형을 주어서 넣고 싶다면 createAction의 두번째 파라미터에 payload를 정의하는 함수를 따로 선언해서 넣어주면 된다.
-> 첫번째 파라미터는 type 값
-> payload를 정의하는 함수의 매개변수 : 넣고싶은 값, 리턴값 : payload에 해당하는 값

const MY_ACTION = "sample/MY_ACTION"
const myAction = createAction("MY_ACTION", text => `${text}!!!`);
const action = myAction('hello');

action => {
    type : My_ACTION,
    payload : 'hello!!!',
}

ex3)
export const toggle = createAction(TOGGLE, (id) => id);

action => {
    type : TOGGLE,
    payload: id
}

** handleAction 함수의 첫 번째 파라미터에는 각 액션에 대한 업데이트 함수를 넣어주고 두번째 파라미터에는 초기 상태를 넣어준다.


2) immer 활용

- 오히려 더 헷갈림
- 객체 depth가 3단계 이상일 때 사용해볼 것


****** 중요하다고 생각되는 것

1. 리덕스 포함한 코드 작성 순서

0) 리덕스 구조 생각하기
1) app.js 에서 store 생성, provider 추가
2) 모듈 작성 : 액션,state 무엇이 필요할까 고민 -> 액션 정의 -> 액션 생성함수 정의(파라미터 뭐가 들어가야할까 고민) -> 리듀서 작성
3) 컨테이너 작성 -> state, dispatch 흐름 잘 파악하기(connect 이용)
4) 컴포넌트 작성 -> state와 이벤트 관계 확실히 규명


2. 주의사항

onChange={(e) => {
    onInputChange(e.target.value); --> action 파라미터가 필요할 때
}}

onChange = {onInputChange} --> action 파라미터가 필요 없을 때

위 두개 코드는 다른 것임 -> 파라미터를 넣고 싶으면 위에 거 써야함. 근데 액션에 payload 값을 넣는 경우가 대부분이므로 위에 것이 훨씬 많음.

****** redux-actions의 모듈 handleAction이 아니고 handleActions임.!!!!!!!!


3. 컨테이너의 connect는 컴포넌트에서 이벤트가 발생했을 시 액션생성 + dispatch까지 함께 해주는 역할을 한다.



***** redux-hook 사용하기

- 리덕스 스토어와 연동된 컨테이너 컴포넌트를 만들 때 connect함수를 사용하는 대신 react-redux에서 제공하는 hooks를 사용할 수도 있다.

* useSelector로 상태 조회하기
- const 결과 = useSelector(상태선택함수);

=> mapStateToProps와 형태가 똑같다.

* useDispatch로 액션생성함수 조회하기



***** connect함수를 사용하여 컨테이너 컴포넌트를 만들었을 경우 해당 컨테이너 컴포넌트의 부모 컴포넌트가 리렌더링될 때 해당 컨테이너 컴포넌트의 props가 바뀌지 않았다면 리렌더링이 자동으로 방지되어 성능이 최적화된다.

반면 useSelector를 사용하여 리덕스 상태를 조회했을 때는 이 최적화 작업이 자동으로 이루어지지 않으므로 성능 최적화를 위해서는 React.memo를 컨테이너 컴포넌트에 사용해주어야 한다.


```
