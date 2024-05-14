# state 일괄처리(batching)에 대해서!

> ✏️ Batching 이란? <br />
> state 값이 변경되었을 경우 React 에서는 해당 컴포넌트를 리렌더링 하며, 불필요한 리렌더링을 방지하기 위해 state를 변경하는 작업을 일괄적으로 처리한다.
> 이렇게 state 의 업데이트 작업을 모아 일괄 처리하는 방식을 Batching 이라고 하며, 이 덕에 React 에서는 불필요한 리렌더링을 방지할 수 있게 되었다.

<br />

## 상태 일괄처리의 동작

리액트에서의 일괄처리는 주로 setState와 같은 상태 업데이트 함수가 여러 번 호출될 때, 각각의 호출이 즉각적인 리렌더링을 유발하지 않도록 하기 위해 사용되는데, 대신, 리액트는 모든 상태 업데이트를 '배치(batch)'로 그룹화하여 한 번의 리렌더링 프로세스에서 처리한다.

### 작동 방식

- 이벤트 핸들러 내 일괄처리: 사용자 이벤트가 발생하면, 리액트는 이벤트 핸들러 내에서 실행된 모든 setState 호출을 자동으로 일괄처리한다. 즉, 하나의 이벤트 처리 과정에서 여러 상태 변경이 요청되더라도 실제 DOM 업데이트는 한 번만 일어난다.

```js
import React, { useState } from "react";

function TeamUpdateExample() {
  const [teamName, setTeamName] = useState("team-Palindrome");
  const [teamMembers, setTeamMembers] = useState([]);

  function updateTeam() {
    setTeamName("자유방임팀");
    setTeamMembers(["권성한", "한민지", "최기원", "홍성준", "금서하"]);
  }

  return (
    <div>
      <button onClick={updateTeam}>Update Team</button>
      <h1>Team Name: {teamName}</h1>
      <h2>Team Members:</h2>
      <ul>
        {teamMembers.map((member, index) => (
          <li key={index}>{member}</li>
        ))}
      </ul>
    </div>
  );
}

export default TeamUpdateExample;
```

위의 코드에서 setTeamName 호출하여 teamName 상태를 "team-Palindrome"에서 "자유방임팀"으로 변경하였습니다.
또한 setTeamMembers 호출하여 teamMembers 상태를 새로운 배열 ["권성한", "한민지", "최기원", "홍성준", "금서하"]로 업데이트하였는데 두 상태 변경이 같은 이벤트 핸들러 내에서 발생하였기 때문에, React는 이를 자동으로 일괄처리하여 하나의 리렌더링 사이클에서 처리하였다. 이는 각각의 상태 업데이트 후에 개별적으로 리렌더링을 발생시키지 않고, 모든 상태 변경이 완료된 후 단 한 번의 리렌더링만을 수행하여 불필요한 렌더링을 줄여 성능을 향상 시켰다.

<br />

- 비동기 작업: 리액트 17까지는 비동기 콜백 내에서의 상태 업데이트가 자동으로 일괄 처리되지 않고 각각의 setState 호출 후 즉시 리렌더링이 발생할 수 있었다. 하지만 리액트 18부터는 createRoot를 사용하는 경우, 비동기 콜백에서의 상태 업데이트도 자동으로 일괄 처리되게 되었다.

```js
import React, { useState, useEffect } from "react";
import { createRoot } from "react-dom/client";

function TeamAsyncUpdate() {
  const [teamName, setTeamName] = useState("team-Palindrome");
  const [teamMembers, setTeamMembers] = useState([]);

  useEffect(() => {
    setTimeout(() => {
      setTeamName("자유방임팀");
      setTeamMembers(["권성한", "한민지", "최기원", "홍성준", "금서하"]);
    }, 2000);
  }, []);

  return (
    <div>
      <h1>Team Name: {teamName}</h1>
      <h2>Team Members:</h2>
      <ul>
        {teamMembers.map((member, index) => (
          <li key={index}>{member}</li>
        ))}
      </ul>
    </div>
  );
}

const container = document.getElementById("root");
const root = createRoot(container);
root.render(<TeamAsyncUpdate />);
```

위에서 설명한 바와 같이 React 18 이전 버전에서는 setTimeout 같은 비동기 함수 내부에서의 상태 업데이트가 일괄 처리되지 않았는데, React 18에서는 createRoot를 사용하면 비동기 함수 내에서도 자동으로 상태 업데이트가 일괄 처리될수가 있다. 그렇기 때문에 위의 코드는 setTimeout 내에서 setTeamName과 setTeamMembers가 연속적으로 호출되어, 이 두 상태 변경 사이에 중간 상태로의 불필요한 리렌더링이 발생하지 않고, 최종 상태로만 한 번 리렌더링된다.

> 💡 createRoot란?? <br />
> React 18 이상에서 도입된 새로운 API로, 애플리케이션의 루트 컴포넌트를 생성하고 관리하기 위해 사용된다. 이 API는 기존의 ReactDOM.render() 함수를 대체하여 더 많은 기능과 더 나은 성능을 제공하며, 특히 새로운 "동시성 모드"(Concurrent Mode)를 완전히 활용할 수 있도록 설계되었다.

```js
import { createRoot } from "react-dom/client";

const container = document.getElementById("root");
const root = createRoot(container);

root.render(<App />);
```

위 코드는 createRoot를 사용하여 애플리케이션의 루트 컴포넌트를 DOM에 마운트하는 과정을 보여주는데, createRoot는 첫 번째 인자로 DOM 요소를 받으며, render 메소드를 통해 지정된 React 컴포넌트를 렌더링하게 된다.

이렇게 일괄처리(batching) 기능은 React에서 중요한 성능 최적화 도구이고, 이를 통해 여러 상태 업데이트를 하나의 리렌더링 사이클로 결합하여 처리함으로써, 불필요한 리렌더링을 줄이고 애플리케이션의 반응성을 향상시킬 수 있다. 특히 React 18에서 createRoot를 도입함으로써 비동기 상황에서도 상태 업데이트를 효과적으로 일괄 처리할 수 있게 되었는데, 이러한 변경은 React 개발자들이 애플리케이션의 성능을 더욱 민감하게 조율할 수 있게 해주며, 복잡한 사용자 인터페이스와 상호 작용을 보다 효율적으로 처리할 수 있도록 돕는다.
