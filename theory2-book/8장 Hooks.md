```js

1. useState
-> 함수형 컴포넌트에서 상탯값 관리

2. useEffect
-> 리액트 컴포넌트가 렌더링될 때마다 특정 작업을 수행하도록 설정할 수 있는 hook. 클래스 컴포넌트의 componentDidMount, componentDidUpdate를 합친 형태이다.

2-1) 마운트될 때만 실행하고 싶을 때
-> useEffect(()=>{...},[])

2-2) 특정 값이 업데이트될 때만 실행하고 싶을 때
-> useEffect(()=>{...},[value])




3. useReducer -> react-redux 대신에 사용하는 react 내장 hook




-> 우리가 이전에 만든 사용자 리스트 기능에서의 주요 상태 업데이트 로직은 App 컴포넌트 내부에서 이루어졌었습니다. 상태를 업데이트 할 때에는 useState 를 사용해서 새로운 상태를 설정해주었는데요, 상태를 관리하게 될 때 useState 를 사용하는것 말고도 다른 방법이 있습니다. 바로, useReducer 를 사용하는건데요, 이 Hook 함수를 사용하면 컴포넌트의 상태 업데이트 로직을 컴포넌트에서 분리시킬 수 있습니다. 상태 업데이트 로직을 컴포넌트 바깥에 작성 할 수도 있고, 심지어 다른 파일에 작성 후 불러와서 사용 할 수도 있지요.


ex)
import React, { useReducer } from "react";

const reducer = (state, action) => { // 상태값과 업데이트에 필요한 정보를 담은 액션값을 받아 새로운 상태를 반환하는 함수
    switch (action.type) {
        case "INCREMENT":
            return { value: state.value + 1 }; // 새로운 상태 반환
        case "DECREMENT":
            return { value: state.value - 1 }; // 새로운 상태 반환
        default:
            return state;
    }
};

const App = () => {
    const [state, dispatch] = useReducer(reducer, { value: 0 }); => reducer함수, 현재상태 초기화
    // state : 현재 컴포넌트의 상태값 dispatch : 상태변경함수 트리거

    return (
        <div>
            <p>현재의 카운터 값은 {state.value} 입니다.</p>
            <button
                onClick={() => {
                    dispatch({ type: "INCREMENT" });
                }}
            >
                +1
            </button>
            <button
                onClick={() => {
                    dispatch({ type: "DECREMENT" });
                }}
            >
                -1
            </button>
        </div>
    );
};

export default App;



ex2)
import React, { useReducer } from "react";

const reducer = (state, action) => {
    return {
        ...state,
        [action.name]: action.value,
    };
};

const App = () => {
    const [state, dispatch] = useReducer(reducer, { name: "", nickname: "" });

    const handleChange = (e) => {
        dispatch(e.target);
    };

    return (
        <div>
            <p>{state.name}</p>
            <p>{state.nickname}</p>
            <input
                type="text"
                name="name"
                placeholder="이름"
                onChange={handleChange}
            />
            <input
                type="text"
                name="nickname"
                placeholder="닉네임"
                onChange={handleChange}
            />
        </div>
    );
};

export default App;




ex3) ****** 매우중요 ******


import React, { useState } from "react";
import { useEffect } from "react";

const App = () => {
    const [state, setState] = useState({ inputValue: 0, values: [], avg: 0 });

    useEffect(() => {
        getAvg();
    }, [state.values]);

    const onChange = (e) => {
        setState({ ...state, inputValue: parseInt(e.target.value) });
    };

    const onClick = (e) => {
        state.inputValue === ""
            ? alert("빈칸을 채워주세요")
            : setState({
                  ...state,
                  values: state.values.concat(state.inputValue),
                  inputValue: 0,
              });
        console.log(state.values); // 곧바로 변경내용이 적용되지 않는다...
        //setState는 비동기이기 때문 그럼 언제 상태가 없데이트 되나? : 렌더링 후..
        //즉 변경된 state를 사용하는 logic라면, 처리를 useEffect에서 해야함
    };

    const getAvg = () => {
        console.log(state.values);
        if (state.values.length > 0) {
            let avg =
                state.values.reduce((a, v, i) => a + v, 0) /
                state.values.length;
            setState({ ...state, avg });
        }
    };

    return (
        <div>
            <input type="text" onChange={onChange} value={state.inputValue} />
            <button onClick={onClick}>등록</button>
            <ul>
                {state.values.map((v, i) => (
                    <li key={i}>{v}</li>
                ))}
            </ul>
            <p>평균은 : {state.avg}</p>
        </div>
    );
};

export default App;





4. useMemo, useCallback



* useMemo와 useCallback을 배우기 전에 알아야 하는 것
1) 함수형 컴포넌트는 그냥 함수다. 다시 한 번 강조하자면 함수형 컴포넌트는 단지 jsx를 반환하는 함수이다.

2) 컴포넌트가 렌더링 된다는 것은 누군가가 그 함수(컴포넌트)를 호출하여서 실행되는 것을 말한다. 함수가 실행될 때마다 내부에 선언되어 있던 표현식(변수, 또다른 함수 등)도 매번 다시 선언되어 사용된다.

3) 컴포넌트는 자신의 state가 변경되거나, 부모에게서 받는 props가 변경되었을 때마다 리렌더링 된다. (심지어 하위 컴포넌트에 최적화 설정을 해주지 않으면 부모에게서 받는 props가 변경되지 않았더라도 리렌더링 되는게 기본이다. )

4) 최적화에 사용되는 Memoization
Memoization이란 이전 값을 메모리에 저장해 동일한 계산의 반복을 제거해 빠른 처리를 가능하게 하는 기술 이라고 한다. useMemo, useCallback, React.memo는 모두 이 Memoization을 기반으로 작동한다.


*** useMemo & useCallback
useMemo는? 사용방법을 제외하고는 React.memo와 매우 흡사하다. React.memo가 component의 결과 값을 memoized하여 불필요한 re-rendering을 관리한다면, useMemo는 함수의 결과 값을 memoized하여 불필요한 연산을 관리한다.


* 메모리제이션된 값을 반환한다라는 문장이 핵심이다. 다음과 같은 상황을 상상해보자. 하위 컴포넌는 상위 컴포넌트로부터 a와 b라는 두 개의 props를 전달 받는다. 하위 컴포넌트에서는 a와 b를 전달 받으면 서로 다른 함수로 각각의 값을 가공(또는 계산)한 새로운 값을 보여주는 역할을 한다. 하위 컴포넌트는 props로 넘겨 받는 인자가 하나라도 변경될 때마다 렌더링되는데, props.a 만 변경 되었을 때 이전과 같은 값인 props.b도 다시 함수를 호출해서 재계산해야 할까? 그냥 이전에 계산된 값을 쓰면 되는데!


*******
useMemo : re-연산 방지
useCallback : re-함수 정의 방지


*** 위의 예제에서 useEffect 대신
const avg = useMemo(()=>{getAvg()},[state.values]) 이런식으로
useMemo hook을 사용할 수도 있다.

ex) 이것은 무슨 경우일까? - 일단 컴포넌트는 하나니까 부모컴포넌트 리렌더링으로 인한 자식 컴포넌트 리렌더링은 아닐 것이고... 정답은 이 컴포넌트가 리렌더링될 때마다 getAvg함수가 계속 호출된다. 즉 list가 바뀌지 않고 onChange만 호출되어도 따라서 불필요한 연산을 제거해줘야함.

const getAvg = (numbers) => {
    console.log("평균값 계산중...");
    if (numbers.length === 0) return 0;
    const sum = numbers.reduce((a, b) => a + b);
    return sum / numbers.length;
};

const App = () => {
    const [list, setList] = useState([]);
    const [number, setNumber] = useState("");

    const onChange = (e) => {
        setNumber(e.target.value);
    };

    const onInsert = () => {
        const nextList = list.concat(parseInt(number));
        setList(nextList);
        setNumber("");
    };

    return (
        <div>
            <input type="text" value={number} onChange={onChange} />
            <button onClick={onInsert}>등록</button>
            <ul>
                {list.map((v, i) => (
                    <li key={i}>{v}</li>
                ))}
            </ul>
            <div>
                <b>평균값 : {getAvg(list)}</b>
            </div>
        </div>
    );
};

export default App;


--> useMemo, useCallback 사용 후.

const getAvg = (numbers) => {
    console.log("평균값 계산중...");
    if (numbers.length === 0) return 0;
    const sum = numbers.reduce((a, b) => a + b);
    return sum / numbers.length;
};

const App = () => {
    const [list, setList] = useState([]);
    const [number, setNumber] = useState("");

    const avg = useMemo(() => {
        getAvg(list);
    }, [list]);

    const onChange = useCallback((e) => { --> re-함수 재정의 방지
        setNumber(e.target.value);
    }, []);

    const onInsert = useCallback(() => { --> re-함수 재정의 방지
        const nextList = list.concat(parseInt(number));
        setList(nextList);
        setNumber("");
    }, [number, list]);

    return (
        <div>
            <input type="text" value={number} onChange={onChange} />
            <button onClick={onInsert}>등록</button>
            <ul>
                {list.map((v, i) => (
                    <li key={i}>{v}</li>
                ))}
            </ul>
            <div>
                <b>평균값 : {avg}</b> --> re-연산방지
            </div>
        </div>
    );
};

export default App;





5. useRef




* JavaScript 를 사용 할 때에는, 우리가 특정 DOM 을 선택해야 하는 상황에 getElementById, querySelector 같은 DOM Selector 함수를 사용해서 DOM 을 선택합니다.

리액트를 사용하는 프로젝트에서도 가끔씩 DOM 을 직접 선택해야 하는 상황이 발생 할 때도 있습니다. 예를 들어서 특정 엘리먼트의 크기를 가져와야 한다던지, 스크롤바 위치를 가져오거나 설정해야된다던지, 또는 포커스를 설정해줘야된다던지 등 정말 다양한 상황이 있겠죠. 추가적으로 Video.js, JWPlayer 같은 HTML5 Video 관련 라이브러리, 또는 D3, chart.js 같은 그래프 관련 라이브러리 등의 외부 라이브러리를 사용해야 할 때에도 특정 DOM 에다 적용하기 때문에 DOM 을 선택해야 하는 상황이 발생 할 수 있습니다.

useRef() 를 사용하여 Ref 객체를 만들고, 이 객체를 우리가 선택하고 싶은 DOM 에 ref 값으로 설정해주어야 합니다. 그러면, Ref 객체의 .current 값은 우리가 원하는 DOM 을 가리키게 됩니다.

ex1)


const App = () => {
    const [list, setList] = useState([]);
    const [number, setNumber] = useState("");

    const numInput = useRef(null); --> useRef를 사용하여 ref객체를 만듬

    const avg = useMemo(() => {
        getAvg(list);
    }, [list]);

    const onChange = useCallback((e) => {
        setNumber(e.target.value);
    }, []);

    const onInsert = useCallback(() => {
        const nextList = list.concat(parseInt(number));
        setList(nextList);
        setNumber("");
        numInput.current.focus(); --> 등록한 DOM ref를 이용하여 작업, current는 ref를 심은 DOM 객체를 가리키게 된다. focus()는 DOM API.
    }, [number, list]);

    return (
        <div>
            <input
                type="text"
                value={number}
                onChange={onChange}
                ref={numInput} --> 만든 ref객체를 DOM에 등록
            />
            <button onClick={onInsert}>등록</button>
            <ul>
                {list.map((v, i) => (
                    <li key={i}>{v}</li>
                ))}
            </ul>
            <div>
                <b>평균값 : {avg}</b>
            </div>
        </div>
    );
};

export default App;

ex2) ref를 컴포넌트 로컬변수로도 이용 가능





6. custom hooks

* 컴포넌트를 만들다보면, 반복되는 로직이 자주 발생합니다. 예를 들어서 input 을 관리하는 코드는 관리 할 때마다 꽤나 비슷한 코드가 반복되죠.

이번에는 그러한 상황에 커스텀 Hooks 를 만들어서 반복되는 로직을 쉽게 재사용하는 방법을 알아보겠습니다.

src 디렉터리에 hooks 라는 디렉터리를 만들고, 그 안에 useInputs.js 라는 파일을 만드세요.

커스텀 Hooks 를 만들 때에는 보통 이렇게 use 라는 키워드로 시작하는 파일을 만들고 그 안에 함수를 작성합니다.

커스텀 Hooks 를 만드는 방법은 굉장히 간단합니다. 그냥, 그 안에서 useState, useEffect, useReducer, useCallback 등 내장 Hooks 를 사용하여 원하는 기능을 구현해주고, 컴포넌트에서 사용하고 싶은 값들을 반환해주면 됩니다.


ex) useInput -> 유저 input 값 관리하기

1) hooks/useInput.js
import React, { useReducer } from "react";

const reducer = (state, action) => {
    return {
        ...state,
        [action.name]: action.value,
    };
};

export const useInput = (initialForm) => {
    const [state, dispatch] = useReducer(reducer, initialForm);

    const onChange = (e) => {
        dispatch(e.target);
    };
    return [state, onChange];
};

export default useInput;

2) App.jsx

const App = () => {
    const [input, setInput] = useInput({ name: "", nickname: "" });
    // input 상태관리 with custom hook
    return (
        <div>
            <input
                type="text"
                placeholder="이름"
                name="name"
                onChange={setState}
            />
            <input
                type="text"
                placeholder="닉네임"
                name="nickname"
                onChange={setState}
            />
            <p>{state.name}</p>
            <p>{state.nickname}</p>
        </div>
    );
};

export default App;


ex2)

1)
import { useState, useCallback } from 'react';

function useInputs(initialForm) {
  const [form, setForm] = useState(initialForm);
  // change
  const onChange = useCallback(e => {
    const { name, value } = e.target;
    setForm(form => ({ ...form, [name]: value }));
  }, []);
  const reset = useCallback(() => setForm(initialForm), [initialForm]);
  return [form, onChange, reset];
}

export default useInputs;


2)
import React, { useRef, useReducer, useMemo, useCallback } from 'react';
import UserList from './UserList';
import CreateUser from './CreateUser';
import useInputs from './hooks/useInputs';

function countActiveUsers(users) { --> util에 넣으면 좋음
  console.log('활성 사용자 수를 세는중...');
  return users.filter(user => user.active).length;
}

const initialState = {
  users: [
    {
      id: 1,
      username: 'velopert',
      email: 'public.velopert@gmail.com',
      active: true
    },
    {
      id: 2,
      username: 'tester',
      email: 'tester@example.com',
      active: false
    },
    {
      id: 3,
      username: 'liz',
      email: 'liz@example.com',
      active: false
    }
  ]
};

function reducer(state, action) {
  switch (action.type) {
    case 'CREATE_USER':
      return {
        users: state.users.concat(action.user)
      };
    case 'TOGGLE_USER':
      return {
        users: state.users.map(user =>
          user.id === action.id ? { ...user, active: !user.active } : user
        )
      };
    case 'REMOVE_USER':
      return {
        users: state.users.filter(user => user.id !== action.id)
      };
    default:
      return state;
  }
}

function App() {
  const [{ username, email }, onChange, reset] = useInputs({
    username: '', --> 인풋 상태 관리용
    email: ''
  });
  const [state, dispatch] = useReducer(reducer, initialState); --> 유저 상태 관리용
  const nextId = useRef(4); --> 로컬변수로 이용.

  const { users } = state;

  const onCreate = useCallback(() => {
    dispatch({
      type: 'CREATE_USER',
      user: {
        id: nextId.current,
        username,
        email
      }
    });
    reset();
    nextId.current += 1;
  }, [username, email, reset]);

  const onToggle = useCallback(id => {
    dispatch({
      type: 'TOGGLE_USER',
      id
    });
  }, []);

  const onRemove = useCallback(id => {
    dispatch({
      type: 'REMOVE_USER',
      id
    });
  }, []);

  const count = useMemo(() => countActiveUsers(users), [users]);
  return (
    <>
      <CreateUser
        username={username}
        email={email}
        onChange={onChange}
        onCreate={onCreate}
      />
      <UserList users={users} onToggle={onToggle} onRemove={onRemove} />
      <div>활성사용자 수 : {count}</div>
    </>
  );
}

export default App;











```
