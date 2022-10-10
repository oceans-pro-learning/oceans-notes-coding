## 轻松理解为什么不用Index作为key

https://juejin.cn/post/6844904133430870024#comment

> 让我们一起来看右边的控制台。每次点击第一个li标签，发现dom结构中ul标签以及剩余的所有li标签都发生了闪烁！

```jsx
import React, {Component} from 'react';
class App extends Component {
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
