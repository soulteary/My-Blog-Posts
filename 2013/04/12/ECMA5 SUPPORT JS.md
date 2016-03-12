# ECMA5 SUPPORT JS

有的时候，做一些小东西的时候，真的很希望可以使用ECMA5中的JS方法，但是想到IE`(ver < 10)`不免淡淡的忧桑。

无意浏览代码的时候发现了一个几个看似不错的库，收藏一下，回头慢慢浏览，其实很多细节写法很不错。

`ecma-5.js`

```js
// ecma-5.js
//
// -- kriskowal Kris Kowal Copyright (C) 2009-2010 MIT License
// -- tlrobinson Tom Robinson
// dantman Daniel Friesen
//
// Array
//
if (!Array.isArray) {
    Array.isArray = function(obj) {
        return Object.prototype.toString.call(obj) === "[object Array]" || (obj instanceof Array);
    };
}
if (!Array.prototype.forEach) {
    Array.prototype.forEach = function(block, thisObject) {
        var len = this.length >>> 0;
        for (var i = 0; i < len; i++) {
            if (i in this) {
                block.call(thisObject, this[i], i, this);
            }
        }
    };
}
if (!Array.prototype.map) {
    Array.prototype.map = function(fun /*, thisp*/ ) {
        var len = this.length >>> 0;
        var res = new Array(len);
        var thisp = arguments[1];

        for (var i = 0; i < len; i++) {
            if (i in this) {
                res[i] = fun.call(thisp, this[i], i, this);
            }
        }
        return res;
    };
}
if (!Array.prototype.filter) {
    Array.prototype.filter = function(block /*, thisp */ ) {
        var values = [];
        var thisp = arguments[1];
        for (var i = 0; i < this.length; i++) {
            if (block.call(thisp, this[i])) {
                values.push(this[i]);
            }
        }
        return values;
    };
}
if (!Array.prototype.reduce) {
    Array.prototype.reduce = function(fun /*, initial*/ ) {
        var len = this.length >>> 0;
        var i = 0;

        // no value to return if no initial value and an empty array
        if (len === 0 && arguments.length === 1) throw new TypeError();

        if (arguments.length >= 2) {
            var rv = arguments[1];
        } else {
            do {
                if (i in this) {
                    rv = this[i++];
                    break;
                }
                // if array contains no values, no initial value to return
                if (++i >= len) throw new TypeError();
            } while (true);
        }
        for (; i < len; i++) {
            if (i in this) {
                rv = fun.call(null, rv, this[i], i, this);
            }
        }
        return rv;
    };
}
if (!Array.prototype.indexOf) {
    Array.prototype.indexOf = function(value /*, fromIndex */ ) {
        var length = this.length;
        var i = arguments[1] || 0;

        if (!length) return -1;
        if (i >= length) return -1;
        if (i < 0) i += length;

        for (; i < length; i++) {
            if (!Object.prototype.hasOwnProperty.call(this, i)) {
                continue
            }
            if (value === this[i]) return i;
        }
        return -1;
    };
}

//
// Object
//
if (!Object.keys) {
    Object.keys = function(object) {
        var keys = [];
        for (var name in object) {
            if (Object.prototype.hasOwnProperty.call(object, name)) {
                keys.push(name);
            }
        }
        return keys;
    };
}

//
// String
//
if (!String.prototype.trim) {
    String.prototype.trim = function() {
        return String(this).replace(/^\s\s*/, '').replace(/\s\s*$/, '');
    };
}
```

- [ddr-ecma5.js via @ddrcode](https://github.com/ddrcode/ddr-ecma5/blob/master/js/ddr-ecma5.js)
- [ECMA262-5 via @charliegroll](https://github.com/charliegroll/ECMA262-5/blob/master/missing-in-ie.js)
