# How far will props go down?

Created By: gihyeon lee
Created Time: May 7, 2020 12:06 AM
Description: Review of Redux for 6 months - 1
Last Edited Time: Jul 14, 2020 12:40 AM
Status: Publishing
Tags: Frontend, React, Redux

Thanks for visiting this post.
All feedbacks on this post are always welcome.
Please send your the feedback by lkh5510@gmail.com.

This post isn't talking about a basic of Redux. This post introduces why I use Redux and the process of solving inconvenience while using Redux.

## TL;DR

- Redux was used because the driling `props` was very inconvenient.
- [redux-actions](https://github.com/redux-utilities/redux-actions) was used because reduce the code of action, reducer.
- [Redux Saga](https://redux-saga.js.org/) was used for async tasks in Redux.
- `routine` was implemented to reduce the duplication of OOO_REQUEST, OOO_SUCCESS, ... in transport in Redux.

## Subject

### Motivation

Redux is a state management tool. This is because the motivation to use is inconvenient when the driling `props`.

When you need user infomation in Root component and CompN-M component, if you don't use Redux, you have to pass user information from Root to CompN-M.

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

**However, if you sue Redux, you don't need that. You just use the state stored in the store by connecting it to CompN-M.**

For the above reasons, I came to use Redux and summarized the problems I encountered and how to solve them.

### When creating Action and Reducer, It's annoying to type one more letter.

When I first created `actions.js`, `reducers.js`, they were as follows.

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

But I thought I could reduce the code by implementing the code more abstractly and i modified it using [redux-actions](https://github.com/redux-utilities/redux-actions) as below.

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

The number of lines of code has been greatly reduced. The example is applied to only one action and reducer, but there may be many actions and reducers in the actual application. **If you want to be comfortable, we recommend using redux-actions.**

### Is there any asynchronous processing library that is more convenient than the Redux Thunk?

I used to the Redux Thunk. I like it. Because I can use clearly Promise in Redux by that. **But I needed a library that could easily use debounce, throttle, cancel, so on. So I found [Redux Saga](https://redux-saga.js.org/).**

The features in Redux Saga I mainly use are as follows:

- `takeLatest`

    A function that performs the action called last.

- Debouncing using `delay`

If you want to learn more, take a look at the following [link](https://redux-saga.js.org/).

### I always hate appending __REQUEST, __SUCCESS, ... to action type when get data to backend.

Basically The order when communication from frontend to backend as follows:

1. Start animation about loading.
2. Send a request to backend.
3. Receive a response from backend.
4. Stop animation about loading.
5. Print message about a result(success, failure)

The Action is divided based on the above order.

- OOO_REQUEST
- OOO_SUCCESS
- OOO_FAILURE
- OOO_COMPLETE

If implemented in code, it is as follows.

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

If the REMOVE_USER action is added, SUCCESS will be diffrenet, and the rest will be the same. **In other words, OOO_COMPLETE, OOO_REQUEST and OOO_FAILURE are likely to overlap in almost all logic that communicates with the backend.**

So, I make `[routine](https://gist.github.com/wes5510/b23cd0f3d6875e5c7ce3d28b01ab5a4b)`. It acts as follows.

- **Create action types of REQUEST, SUCCESS, FAILURE, COMPLETE.**
- **Create basic reducers about action types.**
- **Create and call a logic about REQUEST, SUCCESS, FAILURE, COMPLETE when communicates with the backend in saga.**

The code to which `routine` is applied is as follows.

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

Compared to the previous code, the amount of code has decreased considerably.

## Conclusion

If you are creating a react application, try using Redux.

And I think that it is better to reduce duplicate code because side effects always occur. You might think, "Should I deduplicate this code?" But you reduce a lot of duplicate code, you naturally feel that your coding skill is improving, so I think I should aim to reduce duplicate code.

---

**한국어**

[과연 props는 어디까지 내려가는가](https://www.notion.so/props-4c2959719c654396b7b19f0754407dce)

**External Posts**

[DEV](https://dev.to/wes5510/how-far-will-props-go-down-2ab3), [Medium](https://medium.com/@wes5510/how-far-will-props-go-down-bf0eaf1487ed)
