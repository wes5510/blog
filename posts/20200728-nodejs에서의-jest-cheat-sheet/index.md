# NodeJS에서의 Jest Cheat Sheet

- 모듈의 함수 mocking

    `doSomething1()` 함수를 테스트해야할 때, `ctrl.js`의 `doSomething1()`함수가 같은 모듈의 `doSomething2()`함수를 호출하고 있다면 아래와 같이하면 된다.

    - `ctrl.js`

        ```jsx
        exports.doSomething1 = (a) => {
        	...
        	const result = exports.doSomething2();
        };
        ```

    - `ctrl.test.js`

        `jest.fn()`이 있지만 `jest.spyOn()`을 사용하면 모듈이 유무까지 확인할 수 있다.

        ```jsx
        const ctrl = require('./ctrl');

        describe('ctrl', () => {
        	afterEach(() => {
        		/* 
        		 * spyOn를 이용해서 만든 mock 함수가 
        		 * 다른 test에 영향이 가지 않도록
        		 * 원래 모듈로 초기화
        		*/
        		jest.restoreAllMocks();
        	});

        	describe('doSomething1', () => {
        		test('...', () => {
        			const doSomething2Mock = jest
        				.spyOn(ctrl, 'doSomething2')
        				.mockReturnedValue(1);
        			const ret = ctrl.doSomething1();
        			...
        		});
        	});
        });
        ```

- 모듈의 변수 mocking

    `module1.js`의 `PREFIX`변수를 mocking하면 아래와 같이 할 수 있다.

    - `module1.js`

        ```jsx
        exports.PREFIX = 'pre';
        exports.doSomething = a => `${exports.PREFIX}_${a}`;
        ```

    - `module1.test.js`

        ```jsx
        describe('module', () => {
        	describe('doSomething', () => {
        		let oriPrefix;

        		beforeAll(() => {
        			oriPrefix = module1.PREFIX;
        		});
        	
        		afterAll(() => {
        			module1.PREFIX = oriPrefix;
        		});
        		
        		test('test', () => {
        			module1.PREFIX = '1';
        			...
        		});
        	});
        });
        ```

- 비동기(Async/Await) 함수 테스트

    `ctrl.js`의 비동기 함수인 `doSomething()`를 테스트할 때 아래와 같이 하면된다.

    - `ctrl.js`

        ```jsx
        exports.doSomething = async () => {...};
        ```

    - `ctrl.test.js`

        ```jsx
        const ctrl = require('./ctrl');

        describe('ctrl', () => {
        	describe('doSomething', () => {
        		test('...', async () => {
        			const result = await ctrl.doSomething();
        			...
        		});
        	});
        });
        ```

- 콜백 함수(`res.json()`, ...) 테스트

    콜백 함수가 포함된 `ctrl.js`의 `doSomething()`함수를 테스트할 때 아래와 같이 하면된다.

    - `ctrl.js`

        ```jsx
        exports.doSomething = (e) => {
        	e.on('stop', () => {
        		...
        	});
        };
        ```

    - `ctrl.test.js`

        ```jsx
        const ctrl = require('./ctrl');

        describe('ctrl', () => {
        	describe('doSomething', () => {
        		test('...', done => {
        			const e = {
        				on: jest.fn(() => {
        					...
        					done();
        				});
        			};
        			ctrl.doSomething(e);
        		});
        	});
        });
        ```

- `throw new Error()` 테스트

    `ctrl.js`의 `doSomething()`함수가 에러를 throw하는 지 확인할 때 아래와 같이 하면된다.

    - `ctrl.js`

        ```jsx
        exports.doSomething = () => {
        	throw new Error('?');
        };
        ```

    - `ctrl.test.js`

        ```jsx
        const ctrl = require('./ctrl');

        describe('ctrl', () => {
        	describe('doSomething', () => {
        		test('...', () => {
        			expect(() => ctrl.doSomething()).toThrow(Error);
        		});
        	});
        });
        ```

    `doSomething()`함수가 비동기일 경우 에러를 throw하는 지 확인할 때는 아래와 같이 하면된다.

    - `ctrl.js`

        ```jsx
        exports.doSomething = async () => {
        	...
        	throw Error('?');
        };
        ```

    - `ctrl.test.js`

        ```jsx
        const ctrl = require('./ctrl');

        describe('ctrl', () => {
        	describe('doSomething', () => {
        		test('...', async () => {
        			await expect(ctrl.doSomething()).toThrow(Error);
        		});
        	});
        });
        ```
