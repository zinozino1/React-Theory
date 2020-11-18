```js

1. ref는 언제 사용하는 것일까?
- DOM 에 직접 접근해야할 때 사용해야한다.
- 하지만 대부분의 경우 DOM 에 직접 접근하지 않아도 state로 해결 가능한 경우가 많음.
-> 컴포넌트 내부에 상태 값이 존재한다는 것은 컴포넌트 내부적으로 할 수 있는 일이 많다는 것을 뜻한다.DOM 에 직접 접근안해도 된다는 말임

ex) -> state로 해결 가능한 경우
const App = () => {
    const [state, setState] = useState({
        password: "",
        clicked: false,
        validated: false,
    });

    const handleChange = (e) => {
        setState({ ...state, password: e.target.value });
    };

    const handleValidate = (e) => {
        if (state.password === "0000") {
            setState({ ...state, validated: true, clicked: true });
        }
    };

    return (
        <div>
            {state.validated ? (
                <input
                    style={{ backgroundColor: "green" }}
                    type="text"
                    onChange={handleChange}
                />
            ) : (
                <input type="text" onChange={handleChange} />
            )}

            <button onClick={handleValidate}>validate</button>
        </div>
    );
};

export default App;

- 하지만 DOM 에 직접 접근해야하는 경우가 있음
- 예를들어 특정 input에 포커스 주기, 스크롤 박스 조작하기, canvas요소에 그림 그리기 등
- 하지만 우선 먼저 ref를 사용하지 않고도 원하는 기능을 구현할 수 있는지 반드시 고려한 후에 활용하자. 잘못사용하면 스파게티 코드가 되어버린다.

8장 hooks에서 다시 하기로..




```
