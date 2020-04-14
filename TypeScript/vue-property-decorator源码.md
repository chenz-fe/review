## vue-property-decorator 源码阅读

在 vue + ts 项目中，我们一定会用到 `vue-property-decorator` 这个库，`script`中的代码会变成下面这样：

```html
<template>
  <div id="app">
    <HelloWorld msg="this is a msg from app.vue" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import HelloWorld from './components/HelloWorld.vue'

@Component({
  components: {
    HelloWorld
  }
})
export default class App extends Vue {}
</script>
```

通过代码的引用关系，可以发现 `vue-property-decorator` 的实现依赖于 `vue-class-component`，它具备以下几个属性：

- @Component

- @Emit

- @Inject
- @Provice
- @Prop
- @Watch
- @Model
- Mixins

下面我们通过源码来看看，上面这些装饰器都是如何实现的。

### @Component

`Component`是在`vue-class-component`中实现的。

```js
import Component, { mixins } from 'vue-class-component';
```

下面我们来看下`vue-class-component`

node_modules/vue-class-component/lib/index.js

这里定义了`Component`方法，判断了参数`options`类型，如果不是函数类型，需要包一层函数返回`componentFactory`的调用结果。

```js
import { componentFactory, $internalHooks } from './component';
export { createDecorator, mixins } from './util';
function Component(options) {
  	// 这里如果是函数类型，说明 options 就是 继承于 Vue 的 class 函数
    if (typeof options === 'function') {
        return componentFactory(options);
    }
    return function (Component) {
        return componentFactory(Component, options);
    };
}
Component.registerHooks = function registerHooks(keys) {
    $internalHooks.push(...keys);
};
export default Component;
```

下面是我们通常使用`@Component`的方法，这里如果 options 是函数类型，说明options是`App`这个 class 函数，如果不是函数类型，说明options是传入的`{}`配置项。

```js
@Component({
  components: {
    HelloWorld
  }
})
export default class App extends Vue {}
```

再看下`componentFactory`方法做了什么。

node_modules/vue-class-component/lib/component.js

```js
export function componentFactory(Component, options = {}) {
    options.name = options.name || Component._componentTag || Component.name;
    // prototype props.
    const proto = Component.prototype;
  	// 遍历组件原型对象上的每一个属性，根据属性值的类型，处理 hooks，methods，mixins和computed
    Object.getOwnPropertyNames(proto).forEach(function (key) {
        if (key === 'constructor') {
            return;
        }
        // hooks
        if ($internalHooks.indexOf(key) > -1) {
            options[key] = proto[key];
            return;
        }
      	// Object.getOwnPropertyDescriptor 返回组件原型对象上自有属性对应的属性描述符
        const descriptor = Object.getOwnPropertyDescriptor(proto, key);
        if (descriptor.value !== void 0) {
            // methods
            if (typeof descriptor.value === 'function') {
                (options.methods || (options.methods = {}))[key] = descriptor.value;
            }
            else {
                // typescript decorated data
                (options.mixins || (options.mixins = [])).push({
                    data() {
                        return { [key]: descriptor.value };
                    }
                });
            }
        }
        else if (descriptor.get || descriptor.set) {
            // computed properties
            (options.computed || (options.computed = {}))[key] = {
                get: descriptor.get,
                set: descriptor.set
            };
        }
    });
    (options.mixins || (options.mixins = [])).push({
        data() {
            return collectDataFromConstructor(this, Component);
        }
    });
    // decorate options
    const decorators = Component.__decorators__;
    if (decorators) {
        decorators.forEach(fn => fn(options));
        delete Component.__decorators__;
    }
    // find super
    const superProto = Object.getPrototypeOf(Component.prototype);
    const Super = superProto instanceof Vue
        ? superProto.constructor
        : Vue;
    const Extended = Super.extend(options);
    forwardStaticMembers(Extended, Component, Super);
    if (reflectionIsSupported()) {
        copyReflectionMetadata(Extended, Component);
    }
    return Extended;
}
```

node_modules/vue-class-component/lib/data.js

```js
export function collectDataFromConstructor(vm, Component) {
    // override _init to prevent to init as Vue instance
    const originalInit = Component.prototype._init;
    Component.prototype._init = function () {
        // proxy to actual vm
        const keys = Object.getOwnPropertyNames(vm);
        // 2.2.0 compat (props are no longer exposed as self properties)
        if (vm.$options.props) {
            for (const key in vm.$options.props) {
                if (!vm.hasOwnProperty(key)) {
                    keys.push(key);
                }
            }
        }
        keys.forEach(key => {
            if (key.charAt(0) !== '_') {
                Object.defineProperty(this, key, {
                    get: () => vm[key],
                    set: value => { vm[key] = value; },
                    configurable: true
                });
            }
        });
    };
    // should be acquired class property values
    const data = new Component();
    // restore original _init to avoid memory leak (#209)
    Component.prototype._init = originalInit;
    // create plain data object
    const plainData = {};
    Object.keys(data).forEach(key => {
        if (data[key] !== undefined) {
            plainData[key] = data[key];
        }
    });
    if (process.env.NODE_ENV !== 'production') {
        if (!(Component.prototype instanceof Vue) && Object.keys(plainData).length > 0) {
            warn('Component class must inherit Vue or its descendant class ' +
                'when class property is used.');
        }
    }
    return plainData;
}
```

### @Prop

node_modules/vue-property-decorator/lib/vue-property-decorator.js

这里最核心的就是返回一个函数，即属性装饰器，在函数中对`createDecorator`方法做了调用，传入当前类(vue组件)和被装饰的属性作为参数。

```js
/**
 * decorator of a prop
 * @param  options the options for the prop
 * @return PropertyDecorator | void
 */
export function Prop(options) {
    if (options === void 0) { options = {}; }
    return function (target, key) {
        applyMetadata(options, target, key);
        createDecorator(function (componentOptions, k) {
            ;
            (componentOptions.props || (componentOptions.props = {}))[k] = options;
        })(target, key);
    };
}
```

下面我们看看`createDecorator`方法的实现。

node_modules/vue-class-component/lib/util.js

首先通过`target`获取当前组件的构造函数Ctor，判断如果参数`target`是一个函数，则取`target`，如果`target`是组件的实例对象，则取`target.constructor`；

然后往构造函数的`__decorators__`属性中放进传入的函数`factory`。`factory`的实现上面可以看到，就是将装饰器传入的参数`options`挂载在组件配置选项的`props`上。

```js
export function createDecorator(factory) {
    return (target, key, index) => {
        const Ctor = typeof target === 'function'
            ? target
            : target.constructor;
        if (!Ctor.__decorators__) {
            Ctor.__decorators__ = [];
        }
        if (typeof index !== 'number') {
            index = undefined;
        }
        Ctor.__decorators__.push(options => factory(options, key, index));
    };
}
```

### @Model

node_modules/vue-property-decorator/lib/vue-property-decorator.js

`Model`方法和`Prop`方法大同小异，就是增加在组件配置选项的model属性上挂了一个对象，本来`model`就是属性+事件的语法糖，这里很好理解。

```js
/**
 * decorator of model
 * @param  event event name
 * @param options options
 * @return PropertyDecorator
 */
export function Model(event, options) {
    if (options === void 0) { options = {}; }
    return function (target, key) {
        applyMetadata(options, target, key);
        createDecorator(function (componentOptions, k) {
            ;
            (componentOptions.props || (componentOptions.props = {}))[k] = options;
            componentOptions.model = { prop: k, event: event || k };
        })(target, key);
    };
}
```

### @Watch

node_modules/vue-property-decorator/lib/vue-property-decorator.js

```js
/**
 * decorator of a watch function
 * @param  path the path or the expression to observe
 * @param  WatchOption
 * @return MethodDecorator
 */
export function Watch(path, options) {
    if (options === void 0) { options = {}; }
    var _a = options.deep, deep = _a === void 0 ? false : _a, _b = options.immediate, immediate = _b === void 0 ? false : _b;
    return createDecorator(function (componentOptions, handler) {
        if (typeof componentOptions.watch !== 'object') {
            componentOptions.watch = Object.create(null);
        }
        var watch = componentOptions.watch;
        if (typeof watch[path] === 'object' && !Array.isArray(watch[path])) {
            watch[path] = [watch[path]];
        }
        else if (typeof watch[path] === 'undefined') {
            watch[path] = [];
        }
        watch[path].push({ handler: handler, deep: deep, immediate: immediate });
    });
}
```

### @Emit

```js
// Code copied from Vue/src/shared/util.js
var hyphenateRE = /\B([A-Z])/g;
var hyphenate = function (str) { return str.replace(hyphenateRE, '-$1').toLowerCase(); };
/**
 * decorator of an event-emitter function
 * @param  event The name of the event
 * @return MethodDecorator
 */
export function Emit(event) {
    return function (_target, propertyKey, descriptor) {
        var key = hyphenate(propertyKey);
        var original = descriptor.value;
        descriptor.value = function emitter() {
            var _this = this;
            var args = [];
            for (var _i = 0; _i < arguments.length; _i++) {
                args[_i] = arguments[_i];
            }
            var emit = function (returnValue) {
                var emitName = event || key;
                if (returnValue === undefined) {
                    if (args.length === 0) {
                        _this.$emit(emitName);
                    }
                    else if (args.length === 1) {
                        _this.$emit(emitName, args[0]);
                    }
                    else {
                        _this.$emit(emitName, args);
                    }
                }
                else {
                    _this.$emit(emitName, returnValue);
                }
            };
            var returnValue = original.apply(this, args);
            if (isPromise(returnValue)) {
                returnValue.then(function (returnValue) {
                    emit(returnValue);
                });
            }
            else {
                emit(returnValue);
            }
            return returnValue;
        };
    };
}
```

