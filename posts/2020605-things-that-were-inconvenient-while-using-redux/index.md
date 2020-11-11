# Things that were inconvenient while using Redux

This post isn't talking about a basic of Redux.
Before reading this post, it is helpful to read "[How far will props go down?](https://blog.bundles.dev/posts/20200507-how-far-will-props-go-down)".

## TL;DR

- There were some inconveniences when using Redux.
    1. It was difficult to understand with non-intuitive logic
    2. Readability is poor when debugging because there are empty data that is not needed.
- However, I have improved the above discomfort and still use it. This is because using Redux creates patterns and it ensures high productivity and readability in application development.

## Subject

The following explains the inconveniences of using Redux.

### Non-intuitive Logic

The basic Redux code is shown below.

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

If anyone who doesn't know Redux sees the code above, can he/she knows exactly how n increases by 1? Honestly, I didn't understand at first. I didn't understand where the arguments (state, dispatch) of `mapStateToProps`, `mapDispatchToProps` are input and what is input. I thought I should just use it like that.

I thought I lacked understanding, but when I told my co-workers that this was happening, there were quite a few people like me.

### Empty state data, not needed

Suppose you implement a bulletin board with the following requirements.

- A page showing all posts (I'll call it the /posts page)
- A page showing detailed information (title, content, author) of the post (I will call it the /posts/:postID page)

If you use Redux, you can set InitState as below.

- `initState.js`

    ```jsx
    const initState = {
    	posts: [],
    	post: {}
    };

    export default initState;
    ```

However, `posts` are useful only on the /posts page, and not required for the /posts/:postID page. Currently, there are only two pages, but if there are many pages like the admin application and there are few states used for one page, the empty status data will increase.

There was a lot of empty state data that I didn't need, making it inconvenient to debug in [NEXT.js](https://nextjs.org/) + [Redux DevTools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en), and poor readability.

## Conclusion

Redux has its advantages, but also its disadvantages. However, the disadvantage of having a large learning curve can be covered by learning, and empty state data that are not needed can be avoided by good state design. And if you use Redux well, you can build your application more easily. For example, the pattern that implements business logic in the container and the part that communicates with the backend is implemented in the slice module, so that other developers can code predictably, thereby ensuring high readability and productivity.

---

**한국어**

[Redux를 사용하면서 아쉬웠던 것들](https://blog.bundles.dev/posts/20191020-redux를-사용하면서-아쉬웠던-것들)

**External Posts**

[DEV](https://dev.to/wes5510/things-that-were-inconvenient-while-using-redux-2cl3)
