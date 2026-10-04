# qs

Run tests: `npx tape 'test/**/*.js'`

## Stringifying with a filter function

When `stringify` is called with a `filter` function, it is invoked once for the
root object and once for every value encountered while walking the object. Each
call receives three arguments:

1.  `prefix` — the fully assembled prefix string for the current item. Its shape
    depends on options such as `allowDots`, `arrayFormat`, `encodeDotInKeys`,
    and `encodeValuesOnly` (for example `user[password]`, `user.password`, or
    `users[0][password]`).
2.  `value` — the value at the current item. Returning `undefined` removes that
    item from the output, just as before.
3.  `path` — an array listing every key from the root down to the current item.
    Object keys are strings and array indices are numbers; the root call
    receives an empty array. Unlike `prefix`, `path` does not depend on
    `allowDots`/`arrayFormat`/`encodeDotInKeys`/`encodeValuesOnly`: the same
    location always gets the same array, regardless of how the prefix is
    rendered. Keys that themselves contain dots, brackets, or percent signs
    appear in the array verbatim — never split, escaped, or encoded.

````js
var qs = require('./');

var maskSecrets = function (prefix, value, path) {
    if (path[path.length - 1] === 'password') {
        return '***';
    }
    return value;
};

qs.stringify(
    { user: { name: 'alice', password: 'secret' } },
    { allowDots: true, filter: maskSecrets }
);
// 'user.name=alice&user.password=%2A%2A%2A'
````

The third argument is optional: filter functions written to accept only the
first two arguments keep working unchanged. Passing `filter` as an array is
unaffected as well.
