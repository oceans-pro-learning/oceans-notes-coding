
## componentDidUpdate
视图中只用到了name，没有用到age，而当age发生改变时，控制台仍然会打印出`render App`，如果这是我们不能接受的，那么可以使用`componentDidUpdate`。

```jsx
import React, {Component} from 'react';

class App extends Component {
    constructor() {
        super();
        this.state = {
            name: 'app',
            age: 1,
        };
    }
    // try comment/un-comment this func
    componentDidUpdate(nextProps, nextState) {
        if (nextState.name !== this.state.name) {
            return true;
        }
        return false;
    }

    render() {
        console.log('render App');
        return (
            <div>
                {this.state.name}

                <button
                    onClick={() => {
                        this.setState({
                            name: this.state.name + '+',
                        });
                    }}
                >
                    改变name
                </button>

                <button
                    onClick={() => {
                        this.setState({
                            age: this.state.age + 1,
                        });
                    }}
                >
                    改变age
                </button>
            </div>
        );
    }
}
export default App;
```

## 轻松理解为什么不用Index作为key

https://juejin.cn/post/6844904133430870024#comment

> 让我们一起来看右边的控制台。每次点击第一个li标签，发现dom结构中ul标签以及剩余的所有li标签都发生了闪烁！

```jsx
import React from 'react';
class App extends React.Component {
    constructor(props) {
        super(props);
        this.state = {list: ['a', 'b', 'c', 'd', 'e']};
    }
    render() {
        let list = this.state.list;
        return (
            <ul>
                {list.map((item, index) => (
                    <li
                        key={index}
                        onClick={this.deleteItem.bind(this, index)}
                        style={{fontSize: 30}}
                    >
                        {item}
                    </li>
                ))}
            </ul>
        );
    }
    // 删除点击的item
    deleteItem(index) {
        let newlist = [...this.state.list];
        newlist.splice(index, 1);
        this.setState(() => ({list: newlist}));
    }
}

export default App;
```

## 源码分析

核心源码

```js
// react-16.8.6/packages/react-reconciler/src/ReactFiberClassComponent.js
function checkShouldComponentUpdate(
  workInProgress,
  ctor,
  oldProps,
  newProps,
  oldState,
  newState,
  nextContext,
) {
  const instance = workInProgress.stateNode;
  if (typeof instance.shouldComponentUpdate === 'function') {
    startPhaseTimer(workInProgress, 'shouldComponentUpdate');
    const shouldUpdate = instance.shouldComponentUpdate(
      newProps,
      newState,
      nextContext,
    );
    stopPhaseTimer();

    if (__DEV__) {
      warningWithoutStack(
        shouldUpdate !== undefined,
        '%s.shouldComponentUpdate(): Returned undefined instead of a ' +
          'boolean value. Make sure to return true or false.',
        getComponentName(ctor) || 'Component',
      );
    }

    return shouldUpdate;
  }

  if (ctor.prototype && ctor.prototype.isPureReactComponent) {
    return (
      !shallowEqual(oldProps, newProps) || !shallowEqual(oldState, newState)
    );
  }

  return true;
}
```

PureComponent会在原型上增加isPureReactComponent这一个属性。

```js
// react-16.8.6/packages/react/src/ReactBaseClasses.js
function ComponentDummy() {}
ComponentDummy.prototype = Component.prototype;

/**
 * Convenience component with default shallow equality check for sCU.
 */
function PureComponent(props, context, updater) {
  this.props = props;
  this.context = context;
  // If a component has string refs, we will assign a different object later.
  this.refs = emptyObject;
  this.updater = updater || ReactNoopUpdateQueue;
}

const pureComponentPrototype = (PureComponent.prototype = new ComponentDummy());
pureComponentPrototype.constructor = PureComponent;
// Avoid an extra prototype jump for these methods.
Object.assign(pureComponentPrototype, Component.prototype);
pureComponentPrototype.isPureReactComponent = true;

export {Component, PureComponent};
```

浅层比较的源码

```js
// react-16.8.6/packages/shared/shallowEqual.js
import is from './objectIs';

const hasOwnProperty = Object.prototype.hasOwnProperty;

/**
 * Performs equality by iterating through keys on an object and returning false
 * when any key has values which are not strictly equal between the arguments.
 * Returns true when the values of all keys are strictly equal.
 */
function shallowEqual(objA: mixed, objB: mixed): boolean {
  if (is(objA, objB)) {
    return true;
  }

  if (
    typeof objA !== 'object' ||
    objA === null ||
    typeof objB !== 'object' ||
    objB === null
  ) {
    return false;
  }

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  if (keysA.length !== keysB.length) {
    return false;
  }

  // Test for A's keys different from B.
  for (let i = 0; i < keysA.length; i++) {
    if (
      !hasOwnProperty.call(objB, keysA[i]) ||
      !is(objA[keysA[i]], objB[keysA[i]])
    ) {
      return false;
    }
  }

  return true;
}
```

