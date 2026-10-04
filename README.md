# qs

Run tests: `npx tape 'test/**/*.js'`

## `filter` option for `stringify`

When `filter` is a function, it is called once for the root object and once for
every value visited during serialization. It receives three arguments:

-   `prefix` — the fully assembled key prefix as a string. Its shape depends on
    `allowDots`, `arrayFormat`, `encodeDotInKeys`, and `encodeValuesOnly` (for
    example `user[password]`, `user.password`, `users[0][password]`, or
    `users[][password]`).
-   `value` — the value at the current position. Returning `undefined` removes
    that item, exactly as before.
-   `path` — an array listing every key from the root down to the current item.
    Object keys are strings and array indices are numbers; the root call
    receives an empty array. Unlike `prefix`, `path` is identical regardless of
    the `allowDots`, `arrayFormat`, `encodeDotInKeys`, and `encodeValuesOnly`
    options, and key names containing dots, brackets, or percent signs are
    included verbatim — neither escaped nor split.

```js
var assert = require('assert');
var qs = require('./');

var paths = [];
qs.stringify(
    { users: [{ name: 'a', password: 'secret' }], 'a.b': { password: 's2' } },
    {
        arrayFormat: 'brackets',
        filter: function (prefix, value, path) {
            paths.push(path);
            if (path[path.length - 1] === 'password') {
                return '***';
            }
            return value;
        }
    }
);

assert.deepEqual(paths, [
    [],
    ['users'],
    ['users', 0],
    ['users', 0, 'name'],
    ['users', 0, 'password'],
    ['a.b'],
    ['a.b', 'password']
]);
```

Callbacks that only declare the first two parameters keep working unchanged,
and the array form of `filter` is unaffected.
