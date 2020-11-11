# 과연 props는 어디까지 내려가는가

이 글은 Redux의 기초는 다루지 않습니다. Redux을 사용한 동기부터 사용하면서 불편한 점을 해결한 과정을 소개합니다.

## TL;DR

- 다른 컴포넌트로 `state`를 props 로 넘겨주는 게 매우 불편해서 Redux를 도입
- action, reducer의 코드를 줄이기 위해 [redux-actions](https://github.com/redux-utilities/redux-actions)를 사용
- Redux 비동기 처리 라이브러리로 [Redux Saga](https://redux-saga.js.org/)를 사용
- Redux에서 백엔드 통신의 OOO_REQUEST, OOO_SUCCESS, ...의 중복을 제거하기 위해 `routine`를 구현하여 사용

## 본론

### 사용 계기

Redux는 `state`를 관리하는 도구이다. 사용하게 된 계기는 **다른 컴포넌트로 `state`를 넘겨줄 때, `state`의 값을 다른 컴포넌트의 `props`로 전달하는 게 귀찮아지는 상황이 오기 때문이다.**

Root 컴포넌트와 CompN-M 컴포넌트에서 사용자 정보가 필요할 때 Redux를 안쓴다면 Root에서 CompN-M까지 사용자 정보를 `props`로 전달해야 한다.

```
          --------
          | Root |
          --------
             |
    |--------|--------|
    |                 |
-----------      -----------
| Comp1-1 |      | Comp2-2 |
-----------      -----------
     |
     |
    ...
-----------
| CompN-M |
-----------
```

**하지만 Redux를 쓴다면 그럴 필요 없이 store에 저장돼있는 state를 CompN-M에 connect하여 쓰면 된다.**

위와 같은 이유로 Redux를 사용하게 되었고 사용하면서 겪은 문제와 그 해결 방법을 정리해봤다.

### Action, Reducer 만들때 한글자라도 더 타이핑하기 귀찮다

Redux를 처음 만들때는 `actions.js`, `reducers.js`파일은 아래와 같았다.

- `actions.js`

    ```jsx
    import actionTypes from './types';

    export default {
    	add: user => ({
    		type: actionTypes.ADD_USER
    		user
    	})
    };
    ```

- `reducers.js`

    ```jsx
    import actionTypes from './types';

    const reducer = (state = [], action) => {
      switch (action.type) {
        case actionTypes.ADD_USER:
          return {
    				users: [
    	        ...state.users,
    	        action.user
    	      ]
    			};
        default:
          return state;
      }
    }

    export default reducer;
    ```

하지만 더 추상적으로 구현하여 코드를 줄일 수 있다고 판단이 되었고 [redux-actions](https://github.com/redux-utilities/redux-actions)를 사용하여 아래와 같이 수정했다.

- `actions.js`

    ```jsx
    import { createAction } from 'redux-actions';
    import actionTypes from './types';

    export default {
    	add: createAction(actionTypes.ADD_USER)
    };
    ```

- `reducers.js`

    ```jsx
    import { handleActions } from 'redux-actions';
    import actionTypes from './types';

    const reducer = handleActions({
    	[actionTypes.ADD_USER]: (state, action) => ({
    		users: [ ...state.users, action.payload ]
    	})
    }, { users: [] });

    export default reducer;
    ```

어느 정도 코드 줄 수가 줄어들었다. 예시는 1개의 action, reducer에만 적용했지만, 실제 어플리케이션에는 수많은 action과 reducer가 있을 수 있다. **만약 손을 편하게 해주고 싶다면 redux-actions를 사용하길 권장한다.**

### Redux Thunk보다 더 편하게 쓸만한 비동기 처리 라이브러리가 없을까

이전에 Redux Thunk를 사용했다. 주로 사용하는 `Promise`를 `Redux`에 직관적으로 사용할 수 있어 좋았다. **하지만 debounce, throttle, ajax cancel, ...을 사용하고 싶은 상황이 있었고 그걸 쉽게 사용할 수 있는 라이브러리가 필요했다. 그래서 찾은 게 [Redux Saga](https://redux-saga.js.org/) 였다.**

내가 주로 쓰는 Redux Saga의 기능은 아래와 같다.

- `takeLatest`

    마지막에 호출된 Action을 수행하는 함수

- `delay`을 이용한 Debouncing

더 자세히 알아보고 싶다면 다음 [링크](https://redux-saga.js.org/)를 살펴보기 바란다.

### 백엔드에서 가져올 때 항상 action type에 __REQUEST, __SUCCESS, ...을 붙이기 귀찮다

기본적으로 프론트 엔드에서 백엔드로 요청을 할 때 순서는 아래와 같다.

1. Loading관련 애니매이션 실행
2. 백엔드에 Request
3. 백엔드에서 Response
4. Loading관련 애니매이션 중지
5. 결과(성공, 실패)에 대한 메시지 출력

위의 순서를 기준으로 Action을 나누면 아래와 같다.

- OOO_REQUEST
- OOO_SUCCESS
- OOO_FAILURE
- OOO_COMPLETE

만약 코드를 구현하면 아래와 같다.

- `sagas.js`

    ```jsx
    import axios from 'axios'
    import { takeLatest, put } from 'redux-saga/effects';

    import actionType from './types';

    function* updateUser({ payload }) {
    	let res;
    	try {
    		yield put({ type: actionType.UPDATE_USER_REQUEST });
    		res = yield call(axios.put, '/api/user', { ...payload });
    		yield put({
    			type: actionType.UPDATE_USER_SUCCEESS,
    			payload: res.data,
    		});
    	} catch (err) {
    		yield put({
    			type: actionType.UPDATE_USER_FAILURE,
    			payload: err,
    		});
    	} finally {
    		yield put({
    			type: actionType.UPDATE_USER_COMPLETE
    		});
    	}
    }

    takeLatest(actionType.UPDATE_USER, updateLectureInfo),
    ```

- `reducers.js`

    ```jsx
    import { handleActions } from 'redux-actions';
    import actionType from './types';

    export default handleActions({
    	[actionType.UPDATE_USER_REQUEST]: state => ({
    		...state,
    		loading: {
    			...state.loading,
    			updateUser: true
    		}
    	}),
    	[actionType.UPDATE_USER_SUCCESS]: (state, { payload }) => ({
    		...state,
    		user: payload,
    	}),
    	[actionType.UPDATE_USER_FAILURE]: (state, { payload }) => ({
    		...state,
    		error: {
    			...state.error,
    			updateUser: payload
    		},
    	}),
    	[actionType.UPDATE_USER_COMPLETE]: (state, { payload }) => ({
    		...state,
    		loading: {
    			...state.loading,
    			updateUser: false
    		}
    	})
    });
    ```

만약 REMOVE_USER action이 추가된다면? 위 코드에서 보듯이 SUCCESS만 다르고 나머지는 똑같을 것이다. **즉, OOO_COMPLETE, OOO_REQUEST, OOO_FAILURE는 백엔드와 통신하는 거의 모든 로직에서 중복될 가능성이 높다.**

그래서 만든게 `[routine](https://gist.github.com/wes5510/b23cd0f3d6875e5c7ce3d28b01ab5a4b)`이다. `**routine`은 아래의 역할을 한다.**

- **REQUEST, SUCCESS, FAILURE, COMPLETE action type 생성**
- **action type에 대한 기본적인 reducer 생성**
- **saga에서 백엔드 통신시 REQUEST, SUCCESS, FAILURE, COMPLETE에 대한 로직 생성 및 호출**

`routine`를 적용한 코드는 아래와 같다.

- `routines.js`

    ```jsx
    import _camelCase from 'lodash/camelCase';

    import createRoutine from '../utils/routine';

    const createRoutineWithNamespace = type =>
    	createRoutine('EXAMPLE_NAMESPACE', type);

    export default {
    	updateUser: createRoutineWithNamespace('UPDATE_USER'),
    };
    ```

- `sagas.js`

    ```jsx
    import axios from 'axios'
    import { takeLatest, call } from 'redux-saga/effects';

    import routines from './routines';
    import actionType from './types';

    function* updateUser({ payload }) {
    	yield call(
    		routines.updateUser.action,
    		axios.put,
    		'/api/user',
    		{...payload},
    	);
    }

    takeLatest(actionType.UPDATE_USER, updateLectureInfo),
    ```

- `reducers.js`

    ```jsx
    import { handleActions } from 'redux-actions';

    import { getAllReducerInRoutines } from '../utils/routine';
    import initState from './initState';
    import routines from './routines';

    export default handleActions(
    	{
    		...getAllReducerInRoutines(routines),
    		...routines.updateUser.success.reducer((draft, { payload }) => {
    			draft.user = payload;
    		}),
    	},
    	initState,
    );
    ```

이전 코드와 비교하여 꽤 많은 코드의 양이 줄어들었다.

## 결론

만약 React로 어플리케이션을 만드는 중이라면 한 번쯤 Redux를 써보는 게 좋을 것 같다.

그리고 중복 코드는 항상 사이드 이팩트가 발생하니 반복되는 패턴을 찾아 줄이는게 좋다고 생각한다. "굳이 이 코드를 중복 제거해야하나?"라고 생각할 수도 있지만 많은 중복 코드를 제거하다보면 자연스럽게 자신의 코딩 스킬이 향상하는 걸 느껴서 중복 코드 제거는 지향해야한다고 생각한다.

---

**English**

[How far will props go down?](https://blog.bundles.dev/posts/20200507-how-far-will-props-go-down/)

**외부 포스트**
[DEV](https://dev.to/wes5510/props-40d6), [Medium](https://medium.com/@wes5510/%EA%B3%BC%EC%97%B0-props%EB%8A%94-%EC%96%B4%EB%94%94%EA%B9%8C%EC%A7%80-%EB%82%B4%EB%A0%A4%EA%B0%80%EB%8A%94%EA%B0%80-63ce30ede2b0)
