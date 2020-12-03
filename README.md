# React Immer

## Immer
immer를 사용하여 불변성을 유지하는 작업을 매우 간단하게 처리할 수 있습니다. 이 라이브러리의 사용법은 다음과 같습니다.

```js
import produce from 'immer';
const nextState = produce(originalState, draft => {
	//바꾸고 싶은 값 바꾸기
  draft.somewhere.deep.inside = 5;
})
```
produce라는 함수는 두 가지 파라미터를 받습니다. 첫 번째 파라미터는 수정하고 싶은 상태이고, 두 번째 파라미터는 상태를 어떻게 업데이트할지 정의하는 함수입니다.

두 번째 파라미터로 전달되는 함수 내부에서 원하는 값을 변경하면, produce 함수가 불변성 유지를 대신해 주면서 새로운 상태를 생성해 줍니다.

immer를 사용하여 컴포넌트 상태를 작성할 때는 객체 안에 있는 값을 직정 수정하거나, 배열에 직접적인 변화를 일으키는 push, splice 등의 함수를 사용해도 무방합니다.

<br />

## useState의 함수형 업데이트와 immer 함께 쓰기
~~~js
const [number, setNumber] = useState(0);
//prevNumbers는 현재 number 값을 가리킵니다.\
const onIncrease = useCallback(
  () => setNumber(prevNumber => prevNumber + 1),
  [],
);
~~~

<br />

immer에서 제공하는 produce 함수를 호출할 때, 첫 번째 파라미터가 함수 형태라면 업데이트 함수를 반환합니다.

<br />

~~~js
//  예제
import React, { useCallback, useRef, useState } from "react";
import produce from "immer";

const App = () => {
  const nextId = useRef(1);
  const [form, setForm] = useState({ name: "", username: "" });
  const [data, setData] = useState({
    array: [],
    uselessValue: null,
  });

  //input 수정을 위한 함수
  const onChange = useCallback((e) => {
    const { name, value } = e.target;
    setForm(
      produce((draft) => {
        draft[name] = value;
      })
    );
  }, []);

  //form 등록을 위한 함수
  const onSubmit = useCallback(
    (e) => {
      e.preventDefault();
      const info = {
        id: nextId.current,
        name: form.name,
        username: form.username,
      };

      // array에 새 항목 등록
      setData(
        produce((draft) => {
          draft.array.push(info);
        })
      );

      //form초기화
      setForm({
        name: "",
        username: "",
      });
      nextId.current += 1;
    },
    [data, form.name, form.username]
  );

  // 항목을 삭제하는 함수
  const onRemove = useCallback((id) => {
    setData(
      produce((draft) => {
        draft.array.splice(
          draft.array.findIndex((info) => info.id === id),
          1
        );
      })
    );
  }, []);

  return (
    <div>
      <form onSubmit={onSubmit}>
        <input
          name="username"
          placeholder="아이디"
          value={form.username}
          onChange={onChange}
        />
        <input
          name="name"
          placeholder="이름"
          value={form.name}
          onChange={onChange}
        />
        <button type="submit">등록</button>
      </form>
      <div>
        <ul>
          {data.array.map((info) => (
            <li key={info.id} onClick={() => onRemove(info.id)}>
              {info.username}({info.name})
            </li>
          ))}
        </ul>
      </div>
    </div>
  );
};

export default App;
~~~