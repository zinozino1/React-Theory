```js

1. 객체의 구조가 깊어진다면 불변성을 유지하면서 이를 업데이트 하는 것이 매우 힘들다.

ex) inside의 값을 4로 변경하기

const obj = {
    somewhere : {
        deep : {
            inside : 3,
            array : [1,2,3]
        }
        bar: 3,
    }
    foo : 1
};

const newObj = {
    ...obj,
    somewhere : {
        ...object.somewhere,
        deep: {
            ...object.somewhere.deep
            inside : 4
        }
    }
}


*** immer 사용법
import produce from "immer";

const original = [
    { name: "asdf", username: "2", checked:false },
    { name: "ff", username: "123", checked:true }
]

첫번째 인자 - originalState : 수정하고싶은 상태
두번째 인자 - 상태변경 logic
두번째 인자의 draft - originalState를 가리키는 참조변수

const nextState = produce(originalState, draft => {

    // id가 2인 항목의 checked 값을 true로 설정
    const user = draft.find(t => t.id === 2);
    user.checked = true;

    // 배열에 새로운 데이터 추가
    draft.push( { name: "ff", username: "123", checked:true });

    // id가 1인 항목을 배열에서 삭제
    draft.splice(draft.findIndex(t => t.id === 1), 1);
})



* immer를 사용하지 않은 예제

const Test = () => {

    const nextId = useRef(1); --> 국룰

    const [form, setForm] = useState({ name: "", username: "" });
    const [data, setData] = useState({ array: [], uselessValue: null });

    const onChange = useCallback(
        (e) => {
            const { name, value } = e.target;
            setForm({ ...form, [name]: value });
        },
        [form], --> form이 바뀔때마다 함수 재정의(새 함수가 만들어진다고 생각) 어차피 목적은 렌더링 최적화
    );

    const onSubmit = useCallback(
        (e) => {
            e.preventDefault(); --> 여기서 form은 서버에 데이터 전달 목적이 아닌 input 값을 편하게 받기 위한 수단일뿐
            const info = {
                // 테크닉
                id: nextId.current,
                name: form.name,
                username: form.username,
            };
            setData({ ...data, array: data.array.concat(info) });
            setForm({ ...form, name: "", username: "" });
            nextId.current += 1;
        },
        [data, form.name, form.username],
    );

    const onRemove = useCallback(
        (id) => {
            setData({
                ...data,
                array: data.array.filter((v, i) => {
                    return v.id !== id;
                }),
            });
        },
        [data],
    );

    return (
        <div>
            <form onSubmit={onSubmit}>
                <input
                    type="text"
                    onChange={onChange}
                    name="name"
                    placeholder="이름"
                    value={form.name}
                />
                <input
                    type="text"
                    onChange={onChange}
                    name="username"
                    placeholder="아이디"
                    value={form.username}
                />
                <button type="submit">등록</button>
            </form>
            <div>
                <ul>
                    {data.array.map((v, i) => (
                        <li
                            key={v.id}
                            onClick={() => { -> ***** 중요 - 이벤트 함수에 매개변수를 전달하고 싶으면 이렇게 써야함. ****
                                // 테크닉
                                onRemove(v.id);
                            }}
                        >
                            {v.name}
                        </li>
                    ))}
                </ul>
            </div>
        </div>
    );
};

export default Test;


***** immer + reducer 사용 예제

const formReducer = (state, action) => {
    switch (action.type) {
        case "CHANGE":
            return produce(state, (draft) => {
                if (action.name) {
                    draft.name = action.name;
                } else if (action.username) {
                    draft.username = action.username;
                }
            });
        default:
            return state;
    }
};

const dataReducer = (state, action) => {
    switch (action.type) {
        case "SUBMIT":
            return produce(state, (draft) => {
                draft.array.push(action.info);
            });
        case "REMOVE":
            return produce(state, (draft) => {
                draft.array.splice(
                    draft.array.findIndex((v) => v.id === action.id),
                    1,
                );
            });

        default:
            return state;
    }
};

const Test = () => {
    const nextId = useRef(1);

    const [form, formDispatch] = useReducer(formReducer, {
        name: "",
        username: "",
    });
    const [data, dataDispatch] = useReducer(dataReducer, {
        array: [],
        uselessValue: null,
    });
    const onChange = useCallback((e) => {
        const { name, value } = e.target;
        formDispatch({
            type: "CHANGE",
            [name]: value,
        });
    });

    const onSubmit = useCallback(
        (e) => {
            e.preventDefault();
            const info = {
                // 테크닉
                id: nextId.current,
                name: form.name,
                username: form.username,
            };
            dataDispatch({ type: "SUBMIT", info });

            nextId.current += 1;
        },
        [data, form.name, form.username],
    );
    const onRemove = useCallback(
        (id) => {
            dataDispatch({ type: "REMOVE", id });
        },
        [data],
    );

    return (
        <div>
            <form onSubmit={onSubmit}>
                <input
                    type="text"
                    onChange={onChange}
                    name="name"
                    placeholder="이름"
                    value={form.name}
                />
                <input
                    type="text"
                    onChange={onChange}
                    name="username"
                    placeholder="아이디"
                    value={form.username}
                />
                <button type="submit">등록</button>
            </form>
            <div>
                <ul>
                    {data.array.map((v, i) => (
                        <li
                            style={{ border: "1px solid black" }}
                            key={v.id}
                            onClick={() => {
                                // 테크닉
                                onRemove(v.id);
                            }}
                        >
                            {v.name}
                        </li>
                    ))}
                </ul>
            </div>
        </div>
    );
};

export default Test;



* 결론 : immer를 사용해서 오히려 복잡해질수도 있다. 따라서 불변성을 유지하는 코드가 복잡할 대만 사용해도 충분
하지만 redux에서 매우 유용하게 사용되므로 익숙해지자

```
