# Redux를 사용하면서 아쉬웠던 것들

이 글은 Redux의 기초는 다루지 않습니다.
이 글을 읽기 전 [과연 props는 어디까지 내려가는가](https://blog.bundles.dev/posts/20191021-과연-props는-어디까지-내려가는가)을 읽으면 도움됩니다.

## TL;DR

- Redux를 사용하면서 불편한 점이 있었다.
    1. 직관적이지 않은 로직으로 러닝 커브가 컸다.
    2. 필요하지 않은 빈 상태 데이터가 남아있어서 디버깅시 가독성이 떨어졌다.
- 하지만 위 불편한 점을 커버하기 위한 방법도 있고 Redux를 사용하면 패턴이 생겨 어플리케이션을 개발하는 데 높은 생산성과 가독성을 유지할 수 있기 때문에 아직 사용하고 있다.

## 본론

아래는 Redux를 사용하면서 불편했던 점을 설명한다.

### 직관적이지 않는 로직

Redux는 기본 코드는 아래와 같다.

- `initState.js`

    ```jsx
    const initState = {
    	n: 0
    };

    export default initState;
    ```

- `actions.js`

    ```jsx
    export const add = () => ({
      type: 'ADD'
    });
    ```

- `reducers.js`

    ```jsx
    import initState from './initState';

    const reducers = (state = initState, action) => {
      switch (action.type) {
        case 'ADD':
          return [
            ...state,
            n: state.n + 1
          ];
    		default:
    			return state;
    	}
    }

    export default reducers;
    ```

- `Comp1.jsx`

    ```jsx
    import React from 'react';
    import { add } from '../reducers';

    const Comp1 = ({ n, add }) => 
    	(<div>n<button onClick={() => add()}>+</button></div>);

    const mapStateToProps = state => ({ n: state.n });
    const mapDispatchToProps = dispatch => ({ add: () => dispatch(add()) });
    export default connect(mapStateToProps, mapDispatchToProps)(Comp1);
    ```

만약 Redux를 알지 못하는 사람이 위의 코드을 봤을 때 n이 1증가한다는 흐름을 정확히 알 수 있을까? 솔직히 난 처음에 이해하지 못했다. `mapStateToProps`, `mapDispatchToProps`의 인자(`state`, `dispatch`)가 어디서 입력되고 무엇이 입력되는지 이해하지 못했다. 그저 저렇게 사용해야되구나라고 생각했다.

처음에는 내가 이해력이 부족한 줄 알았지만 직장 동료들에게 이런 일이 있었다고 말하니 나와 같은 사람이 꽤 많았다.

### 필요하지 않은 빈 상태 데이터

만약 아래와 같은 요구사항을 가지고 있는 게시판을 구현한다고 생각하자.

- 모든 게시글을 보여주는 페이지(/posts 페이지라고 칭하겠다)
- 게시글에 대한 상세 정보(제목, 내용, 글쓴이)을 보여주는 페이지(/posts/:postID 페이지라고 칭하겠다)

Redux을 사용한다면 InitState를 아래와 같이 설정할 수 있다.

- `initState.js`

    ```jsx
    const initState = {
    	posts: [],
    	post: {}
    };

    export default initState;
    ```

하지만 posts는 /posts 페이지에서만 쓸모있고 /posts/:postID 페이지에서는 없어도 되는 상태이다. 현재는 2개의 페이지만 있지만 admin 어플리케이션과 같이 페이지는 많고 한 페이지에 사용하는 상태는 적다면 빈 상태 데이터는 많아질 것이다.

필요없는 빈 상태 데이터가 많아 [NEXT.js](https://nextjs.org/) + [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en)에서 디버깅하기 불편했고 가독성도 떨어졌다.

## 결론

Redux는 장점도 있지만 역시 단점도 있다. 하지만 러닝 커브가 크다는 단점은 학습으로 커버할 수 있고 필요하지 않은 빈 상태 데이터는 State 설계를 잘하면 피해갈 수 있다. 그리고 Redux를 잘 사용하면 어플리케이션를 더 쉽게 구현할 수 있다. 예를 들어 container에는 비즈니스 로직을 구현하고 백엔드와 통신하는 부분은 slice 모듈에 구현한다는 패턴이 생겨 다른 개발자들이 예상 가능한 코딩을 할 수 있어 높은 가독성과 생산성을 보장할 수 있다.

---

**English**

[Things that were inconvenient while using Redux](https://blog.bundles.dev/posts/2020605-things-that-were-inconvenient-while-using-redux)

**외부 포스트**

[DEV](https://dev.to/wes5510/redux-1m34)
