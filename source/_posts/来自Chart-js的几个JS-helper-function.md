---
title: 来自Chart.js的几个JS helper function
date: 2017-05-02 13:44:47
tags: [JavaScript]
---
最近兼职的项目里面因为要用charts进行数据的交互式可视化，用Chart.js比较多，也踩了不少坑。
为了改bug提pr外加学习一下提高姿势水平花了一点时间看了下源码，发现一些比较有用简介的[helper function](https://gist.github.com/xg-wang/dabd06d4736844d20b17c55e6d47858e)很值得学习及日常使用。
<!-- more -->
## 代码
```javascript
var helpers = {};

// -- Basic js utility methods
helpers.each = function(loopable, callback, self, reverse) {
	// Check to see if null or undefined firstly.
	var i, len;
	if (helpers.isArray(loopable)) {
		len = loopable.length;
		if (reverse) {
			for (i = len - 1; i >= 0; i--) {
				callback.call(self, loopable[i], i);
			}
		} else {
			for (i = 0; i < len; i++) {
				callback.call(self, loopable[i], i);
			}
		}
	} else if (typeof loopable === 'object') {
		var keys = Object.keys(loopable);
		len = keys.length;
		for (i = 0; i < len; i++) {
			callback.call(self, loopable[keys[i]], keys[i]);
		}
	}
};
helpers.clone = function(obj) {
	var objClone = {};
	helpers.each(obj, function(value, key) {
		if (helpers.isArray(value)) {
			objClone[key] = value.slice(0);
		} else if (typeof value === 'object' && value !== null) {
			objClone[key] = helpers.clone(value);
		} else {
			objClone[key] = value;
		}
	});
	return objClone;
};
helpers.extend = function(base) {
	var setFn = function(value, key) {
		base[key] = value;
	};
	for (var i = 1, ilen = arguments.length; i < ilen; i++) {
		helpers.each(arguments[i], setFn);
	}
	return base;
};
```

## 使用场景
### `helpers.each`
提供了数组和Object统一的遍历函数，实际使用举例：
```javascript
helpers.each(scalesOptions.xAxes, function(xAxisOptions, index) {
  xAxisOptions.id = xAxisOptions.id || ('x-axis-' + index);
});
```

### `helpers.clone`
提供了完整的深拷贝函数，与常用的`JSON.parse(JSON.stringify(obj))`的区别在于这个函数可以深拷贝对象内的函数。

Chart.js内部用这个进行config之类的merge时，先深拷贝然后再扩展，比较方便。
```javascript
var base = helpers.clone(_base);
```

### `helpers.extend`
Chart.js的设计思想包含了很多plugin形式，本身的Chart的核心功能也都有扩展的方式定义的，关键就在extend。

```javascript
helpers.extend(Chart.prototype, /** @lends Chart */ {
  /**
    * @private
    */
  construct: function(item, config) {
    // ...
  },

  /**
    * @private
    */
  initialize: function() {
    // ...
  },
  // ...
}
```

可以看出直接给原型进行扩展，写起来非常简洁。
