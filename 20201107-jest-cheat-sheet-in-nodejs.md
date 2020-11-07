---
comments: true
---

# Jest Cheat Sheet In NodeJS

- Mocking a function in the module

    When testing the `doSomething1()` function, If the `doSomething1()` function in `ctrl.js` module invokes `doSomething2()` in the same module, you can do as follows.

    - `ctrl.js`

        ```jsx
        exports.doSomething1 = (a) => {
                ...
                const result = exports.doSomething2();
        };
        ```

    - `ctrl.test.js`

        There is `jest.fn()`, but if you use `jest.spyOn()`, you can even check the existence of a function.

        ```jsx
        const ctrl = require('./ctrl');

        describe('ctrl', () => {
                afterEach(() => {
                        /* 
                         * Initialize to the original module 
                         * so that the mock function created using spyOn 
                         * does not affect other tests.
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

- Mocking a variable in the module

    If you are mocking the `PREFIX` variable in `module1.js`, you can do it as follows.

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

- Testing a async function

    When testing the `doSomething()` function in `ctrl.js` that includes the callback function, do as follows.

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

- Testing a callback function(`res.json()`, ...)
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

- Testing `throw new Error()`

    When checking whether the `doSomething()` function in `ctrl.js` throws an error, do as follows.

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

    If `doSomething()` function is asynchronous, you can check whether an error is thrown as follows.

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
