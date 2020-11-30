```js

1. 미들웨어란?

- 리덕스 미들웨어는 액션을 디스패치했을 때 리듀서에서 이를 처리하기에 앞서 사전에 지정된 작업들을 실행한다. 미들웨어는 액션과 리듀서 사이의 중간자라고 볼 수 있다

- 액션 -> 미들웨어 -> 리듀서 -> 스토어

- 리듀서가 액션을 처리하기 전에 미들웨어가 할 수 있는 작업은 여러 가지가 있는데, 전달받은 액션을 단순히 콘솔에 기록하거나, 전달받은 액션 정보를 기반으로 액션을 아예 취소하거나, 다른 종류의 액션을 추가로 디스패치할 수도 있다.

리덕스 미들웨어는 왜 필요해?
기존의 리덕스는 액션이 발생하게 되면, 디스패치를 통해 스토어에게 상태 변화의 필요성을 알리게 됩니다. 하지만, 디스패치된 액션을 스토어로 전달하기 전에 처리하고 싶은 작업이 있을 수 있겠죠. 가령, 단순히 어떤 액션이 발생했는지 로그를 남길 수도 있겠고, 액션을 취소해버리거나, 또 다른 액션을 발생시킬 수도 있습니다.

우리가 알고 있는 리덕스는 동기적인 흐름을 통해 동작합니다. 액션 객체가 생성되고, 디스패치가 액션 발생을 스토어에게 알리면, 리듀서는 정해진 로직에 의해 액션을 처리한 후 새로운 상태값을 반환하는 과정이죠. 하지만, 동기적인 흐름만으로는 처리하기 힘든 작업들이 있습니다. 가령, 시간을 딜레이시켜 동작하게 한다던지, 외부 데이터를 요청하여 그에 따른 응답을 화면에 보여주어야 한다면 어떻게 처리해주어야 할까요?

리덕스에서는 이러한 비동기 작업을 처리하기 위한 지침을 알려주지 않고 있기 때문에 이러한 비동기 작업을 처리하는 데 있어 리덕스 미들웨어를 주로 사용합니다.

리덕스 미들웨어의 동작 방식을 잠시 생각해보자면, 디스패치된 액션이 스토어로 전달되기 전에 미들웨어에서 원하는 작업을 처리한 후 스토어로 액션을 전달한다고 공부했습니다.

*** 외부 API 와 연동하는 작업은 굳이 redux 미들웨어를 사용하지 않아도 좋으나(컴포넌트 내부 state, hook으로 대체 가능) 좋은 개발자 경험이 따라온다.



-> 미들웨어 기본 구조 : 함수를 반환하는 함수를 반환하는 함수
const loggerMiddleware = function (store){
    return function(next){
        return function(action){

        }
    }
}

- store는 리덕스 스토어 인스턴스
- action은 디스패치된 액션을 가리킴
- next 파라미터는 함수 형태이며 store.dispatch와 비슷한 역할을 함 하지만 차이점은 next(action)을 호출하면 그 다음 처리해야 할 미들웨어에게 액션을 넘겨주고, 만약 그 다음 미들웨어가 없다면 리듀서에게 액션을 넘겨준다.


ex) 이렇게 사용 가능.

/lib/loggerMiddleware.js
const loggerMiddleware = (store) => (next) => (action) => {
    console.group(action && store.getState());
    console.log("이전상태", store.getState());
    console.log("액션", action);
    next(action); // 리듀서에서 액션 처리! - 이거 없으면 액션 무시하는 것
    console.log("다음상태", store.getState());
    console.groupEnd();
};

export default loggerMiddleware;

/index.js
const store = createStore(rootReducer, applyMiddleware(loggerMiddleware));

* 미들웨어에서는 여러 종류의 작업을 처리할 수 있다. 특정 조건에 따라 액션을 무시하게 할 수도 있고, 특정 조건에 따라 액션정보를 가로채서 변경한 후 리듀서에게 전달해 줄 수도 있다. 아니면 특정 액션에 기반하여 새로운 액션을 여러 번 디스패치할 수도 있다.

* 이러한 미들웨어 속성을 사용하여 네트워크 요청과 같은 비동기 작업을 관리하면 매우 유용하다.

* 리듀서 함수는 순수함수기 때문에 이 안에서 네트워크 요청을 하면 안된다. 미들웨어에서 처리해야 함



** redux-logger 사용하기
- 방금 만든 loggerMiddleware보다 훨씬 더 잘 만들어진 라이브러리이며 좋음

import { createLogger } from "redux-logger";

const logger = createLogger();
const store = createStore(rootReducer, applyMiddleware(logger));


2. 비동기 작업을 처리하는 미들웨어 사용

***** redux-thunk

- 비동기 작업을 처리할 때 가장 많이 사용하는 미들웨어. 객체가 아닌 함수 형태의 액션을 디스패치할 수 있게 해준다. -> 왜? : 만들어진 액션을 딜레이 시키려고(액션을 가지고 하는 디스패치를 딜레이)

- thunk란? 특정 작업을 나중에 할 수 있도록 미루기 위해 함수형태로 감싼 것을 의미.

- 이 미들웨어를 사용하면 함수를 디스패치 할 수 있다고 했는데, 함수를 디스패치 할 때에는, 해당 함수에서 dispatch 와 getState 를 파라미터로 받아와주어야 한다. 이 함수를 만들어주는 함수를 우리는 thunk 라고 부른다.

- modules에서 액션 생성 함수 다음에 thunk 생성 함수를 만들어서 액션 생성 함수를 딜레이 시킨다.

-**** thunk를 사용하면 action의 type이 function일 경우 next()가 불리지 않는다.
따라서 아래 예제에 액션이 두 번 출력됨

ex)

/modules
export const increase = createAction(INCREASE); --> 내부적으로 store.dispatch가 사용됨
export const decrease = createAction(DECREASE);

export const increaseAsync = () => (dispatch, getState) => { --> action1. 내부적으로 store.dispatch가 사용되지 않는다. 따라서 next를 사용하지 않고, 액션이 리듀서로 전달되지 않는 것
    setTimeout(() => { // middleware : dispatch 내가할께 ^^
        dispatch(increase()); --> action2
    }, 1000);
};

export const decreaseAsync = () => (dispatch, getState) => {
    setTimeout(() => { // middleware : dispatch 내가할께 ^^
        dispatch(decrease());
    }, 1000);
};

/container
const CounterContainer = ({ number, increaseAsync, decreaseAsync }) => {
    return (
        <div>
            <Counter
                number={number}
                onIncrease={increaseAsync}
                onDecrease={decreaseAsync}
            ></Counter>
        </div>
    );
};

export default connect(
    (state) => ({
        number: state.counter,
    }),
    { increaseAsync, decreaseAsync },
)(CounterContainer);





***** redux-saga

- redux-thunk의 경우엔 함수를 디스패치 할 수 있게 해주는 미들웨어였지요? redux-saga의 경우엔, 액션을 모니터링하고 있다가, 특정 액션이 발생하면 이에 따라 특정 작업을 하는 방식으로 사용합니다. 여기서 특정 작업이란, 특장 자바스크립트를 실행하는 것 일수도 있고, 다른 액션을 디스패치 하는 것 일수도 있고, 현재 상태를 불러오는 것 일수도 있습니다.

redux-saga는 redux-thunk로 못하는 다양한 작업들을 처리 할 수 있습니다. 예를 들자면..

- 비동기 작업을 할 때 기존 요청을 취소 처리 할 수 있습니다
- 특정 액션이 발생했을 때 이에 따라 다른 액션이 디스패치되게끔 하거나, 자바스크립트 코드를 실행 할 수 있습니다.
- 웹소켓을 사용하는 경우 Channel 이라는 기능을 사용하여 더욱 효율적으로 코드를 관리 할 수 있습니다 (참고)
- API 요청이 실패했을 때 재요청하는 작업을 할 수 있습니다.
이 외에도 다양한 까다로운 비동기 작업들을 redux-saga를 사용하여 처리 할 수 있답니다.

redux-saga는 다양한 상황에 쓸 수 있는 만큼, 제공되는 기능도 많고, 사용방법도 진입장벽이 꽤나 큽니다. 자바스크립트 초심자라면 생소할만한 Generator 문법을 사용하는데요, 이 문법을 이해하지 못하면 redux-saga를 배우는 것이 매우 어려우니, 이 문법부터 작동방식을 이해해보도록 하겠습니다.



** generator

- 제너레이터 함수를 호출하면 제너레이터 반환
- 제너레이터는 이터러블이자 이터레이터. next()함수 호출가능

주의점
1) yield 보다 next가 1개 더 많음
2) next에 인자 전달할 때는 그 전의 yield 에 값을 넣는 것
3) next 호출시 yield 뒤에 있는 것이 value로 들어감

ex)
function* generator(){
    let a = yield;
    let b = yield;
    yield a + b;
}

const gene = generator();
gene.next(); -> {value : undefined, done: false}
gene.next(1); -> {value : undefined, done: false}
gene.next(2); -> {value : 3, done: false}
gene.next(); -> {value : undefined, done: true}


ex2) -> redux-saga는 이 코드와 비슷한 원리로 작동된다.
function* watchGenerator() {
    console.log("모니터링중..");
    let prevAction = null;
    while (true) {
        const action = yield;
        console.log("이전액션", prevAction);
        prevAction = action;
        if (action.type === "HELLO") {
            console.log("하이여");
        }
    }
}

const watch = watchGenerator();
watch.next();
watch.next({type:"TEST"});
watch.next({type:"HELLO"});

위 예제를

function* watchGenerator() {
    console.log("모니터링중..");
    let prevAction = null;

        const action = yield;
        console.log("이전액션", prevAction);
        prevAction = action;
        if (action.type === "HELLO") {
            console.log("하이여");
        }
        const action = yield;
        console.log("이전액션", prevAction);
        prevAction = action;
        if (action.type === "HELLO") {
            console.log("하이여");
        }
        const action = yield;
        console.log("이전액션", prevAction);
        prevAction = action;
        if (action.type === "HELLO") {
            console.log("하이여");
        }

}

const watch = watchGenerator();
watch.next();
watch.next({type:"TEST"});
watch.next({type:"HELLO"});

이렇게 이해하면 편하다

-> 이 코드가 위의 역할을 수행한다(모니터링)
export function* counterSaga() {
    yield takeEvery(INCREASE_ASYNC, increaseSaga);
    // 들어오는 모든 액션에 대해 특정 작업을 처리해줌
    yield takeLatest(DECREASE_ASYNC, decreaseSaga);
    // 기존에 진행중이던 작업이 있다면 그것을 취소 처리하고 가장 마지막으로 실행된 작업만 수행
}


redux-thunk에서는 thunk 함수를 만들어서 해당 함수에서 비동기 작업을 하고 필요한 시점에 특정 액션을 디스패치합니다. redux-saga는 비동기 작업을 처리 할 때 다른 방식으로 처리합니다.

redux-saga에서는 특정 액션을 모니터링하도록 하고, 해당 액션이 주어지면 이에 따라 제너레이터 함수를 실행하여 비동기 작업을 처리 후 액션을 디스패치합니다.




**** saga 사용법
1) saga 제너레이터 함수 만들기
2) 모니터링 함수 만들기
3) 루트사가 만들기
4) 사가 미들웨어 등록



```
