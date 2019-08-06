# MobX

리액트 상태 관리 라이브러리  
"MobX 는 최소한의 공수로 여러분들의 상태관리 시스템을 설계 할 수 있게 해줍니다."  
<br>

# MobX 라이브러리 설치

```JavaScript
yarn add mobx mobx-react
```


## mobx

**`decorate`**    decorate 를 통해서 각 값에 MobX 함수 적용  
**`observable`**  관찰 받고 있는 상태  
**`action`**  액션; 행동  


## mobx-react

**`observer`**    observer 가 observable 값이 변할 때 컴포넌트의 forceUpdate 를 호출하게 함으로써 자동으로 변화가 화면에 반영  
**`inject`**  mobx-react 에 있는 함수로서, 컴포넌트에서 스토어에 접근할 수 있게 해줍니다.   
**`Provider`**    MobX에서 프로젝트에 스토어를 적용 할 때는, Redux 처럼 Provider 라는 컴포넌트를 사용  


우리가 만약에 create-react-app 으로 프로젝트를 만들면 기본적으로는 decorator 를 사용하지 못하기 때문에 따로 babel 설정을 해줘야 합니다.

<br/>

# MobX - decorate 사용

decorator 를 사용하면 훨씬 더 편하게 문법을 작성 할 수 있는데요, 그러려면 babel 설정을 해주셔야 합니다. babel 설정을 커스터마이징 하려면 yarn eject 를 해야합니다.

### **src/Counter.js**

```JavaScript
import React, { Component } from 'react';
import { decorate, observable, action } from 'mobx';
import { observer } from 'mobx-react';

class Counter extends Component {
  number = 0;

  increase = () => {
    this.number++;
  }

  decrease = () => {
    this.number--;
  }

  render() {
    return (
      <div>
        <h1>{this.number}</h1>
        <button onClick={this.increase}>+1</button>
        <button onClick={this.decrease}>-1</button>
      </div>
    );
  }
}

decorate(Counter, {
  number: observable,
  increase: action,
  decrease: action
})

export default observer(Counter);
```

setState 없이 그냥 값 바꿔주면 알아서 렌더링해줍니다. 어떻게 알아서 렌더링해주냐구요? 
코드 최하단에서 사용된 observer 가 observable 값이 변할 때 컴포넌트의 forceUpdate 를 호출하게 함으로써 자동으로 변화가 화면에 반영됩니다.

이게 성능상으로 과연 좋을까 걱정이 될 수도 있긴 하지만 놀랍게도 최적화가 많이 되어있어서 그 부분에 대해서는 크게 걱정하실 필요없습니다.

이런식으로, 리액트에서 MobX 를 사용할 땐 리덕스에서 했던 것 처럼 따로 다른 파일로 스토어를 만들 필요도 없고 (필요하면 만들 수도 있습니다) 그냥 컴포넌트에 바로 적용해줄 수 있습니다.

<br/>

# Decorator 와 함께 사용하기

decorator 를 사용하면 훨씬 더 편하게 문법을 작성 할 수 있는데요,  
그러려면 babel 설정을 해주셔야 합니다. babel 설정을 커스터마이징 하려면 yarn eject 를 해야합니다.

[Create-react-app V2 릴리즈!](https://velog.io/@velopert/create-react-app-v2)
[create-react-app 에서 eject 명령으로 설정 파일 추출](https://blog.grotesq.com/post/691)

```javascript
yarn eject
```

```javascript
yarn add @babel/plugin-proposal-class-properties @babel/plugin-proposal-decorators
```

그리고 나서, package.json 을 열으신 다음에, babel 쪽을 찾아서 다음과 같이 수정해주세요.

```javascript
"babel": {
  "presets": [
    "react-app"
  ],
  "plugins": [
      ["@babel/plugin-proposal-decorators", { "legacy": true}],
      ["@babel/plugin-proposal-class-properties", { "loose": true}]
  ]
}
```
### **src/Counter.js**

```javascript
import React, { Component } from 'react';
import { observable, action } from 'mobx';
import { observer } from 'mobx-react';

// **** 최하단에 잇던 observer 가 이렇게 위로 올라옵니다.
@observer
class Counter extends Component {
  @observable number = 0;

  @action
  increase = () => {
    this.number++;
  }

  @action
  decrease = () => {
    this.number--;
  }

  render() {
    return (
      <div>
        <h1>{this.number}</h1>
        <button onClick={this.increase}>+1</button>
        <button onClick={this.decrease}>-1</button>
      </div>
    );
  }
}


// **** decorate 는 더 이상 필요 없어집니다.
// decorate(Counter, {
//   number: observable,
//   increase: action,
//   decrease: action
// })

// export default observer(Counter);
// **** observer 는 코드의 상단으로 올라갑니다.
export default Counter;
```
우선 우리가 이 튜토리얼의 상단부에서 다뤘던것처럼 decorator 사용은 필수는 아니라는 점

<br/>

# MobX 스토어 분리시키기

MobX 에도 리덕스처럼 스토어라는 개념이 있습니다.   
리덕스는 하나의 앱에는 단 하나의 스토어만 있지만, MobX 에서는 여러개를 만들어도 됩니다.  

## 스토어 만들기

MobX 에서 스토어를 만드는건 생각보다 간단합니다.   
리덕스처럼 리듀서나, 액션 생성함수.. 그런건 없습니다. 그냥 하나의 클래스에 observable 값이랑 함수들을 만들어주면 끝!  

### **stores/counter.js**

```javascript
import { observable, action } from 'mobx';

export default class CounterStore {
  @observable number = 0;

  @action increase = () => {
    this.number++;
  }

  @action decrease = () => {
    this.number--;
  }
}
```

## Provider 로 프로젝트에 스토어 적용

MobX에서 프로젝트에 스토어를 적용 할 때는, Redux 처럼 Provider 라는 컴포넌트를 사용합니다.  

### **src/index.js**

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'mobx-react'; // MobX 에서 사용하는 Provider
import App from './App';
import CounterStore from './stores/counter'; // 방금 만든 스토어 불러와줍니다.

const counter = new CounterStore(); // 스토어 인스턴스를 만들고

ReactDOM.render(
  <Provider counter={counter}>
    {/* Provider 에 props 로 넣어줍니다. */}
    <App />
  </Provider>,
  document.getElementById('root')
);
```

## inject 로 컴포넌트에 스토어 주입

inject 함수는 mobx-react 에 있는 함수로서, 컴포넌트에서 스토어에 접근할 수 있게 해줍니다. 정확히는, 스토어에 있는 값을 컴포넌트의 props 로 "주입"을 해줍니다.

### **stores/Counter.js**

```javascript
import React, { Component } from 'react';
import { observer, inject } from 'mobx-react';

@inject('counter')
@observer
class Counter extends Component {
  render() {
    const { counter } = this.props;
    return (
      <div>
        <h1>{counter.number}</h1>
        <button onClick={counter.increase}>+1</button>
        <button onClick={counter.decrease}>-1</button>
      </div>
    );
  }
}

export default Counter;
```

위와 같이 inject('스토어이름') 을 하시면 컴포넌트에서 해당 스토어를 props 로 전달받아서 사용 할 수 있게 됩니다.

<br/>

# 스토어의 특정 값이나 함수만 넣어주고 싶다면

마치 리덕스에서의 mapStateToProps / mapDispatchToProps 처럼 스토어의 특정 값이나 함수만 넣어주고 싶다면 이렇게 하실 수도 있습니다.

### **src/Counter.js**

```javascript
import React, { Component } from 'react';
import { observer, inject } from 'mobx-react';

// **** 함수형태로 파라미터를 전달해주면 특정 값만 받아올 수 있음.
@inject(stores => ({
  number: stores.counter.number,
  increase: stores.counter.increase,
  decrease: stores.counter.decrease,
}))
@observer
class Counter extends Component {
  render() {
    const { number, increase, decrease } = this.props;
    return (
      <div>
        <h1>{number}</h1>
        <button onClick={increase}>+1</button>
        <button onClick={decrease}>-1</button>
      </div>
    );
  }
}

export default Counter;
```

이제 컴포넌트는, 유저 인터페이스와, 인터랙션만 관리하면 되고 상태 관련 로직은 모두 스토어로 분리되었습니다.

리덕스에서는, 우리가 프리젠테이셔널 컴포넌트 / 컨테이너 컴포넌트 라는 개념에 대해서 알아보았었습니다. 단순히 props 값을 가져오기만 해서 받아오는 컴포넌트는 프리젠테이셔널 컴포넌트라고 부르고, 스토어에서 부터 값이나 액션 생성함수를 받아오는 컴포넌트를 컨테이너 컴포넌트라고 부른다고 했었죠.

리덕스 진영에서는, 문서에서도 그렇고 생태계 쪽에서도 그렇고 프리젠테이셔널 / 컨테이너로 분리해서 작성하는게 일반적입니다. 반면, MobX 에서는, 딱히 그런걸 명시하지 않습니다. 그래서, 굳이 번거롭게 컨테이너를 강제적으로 만드실 필요는 없습니다. 하지만, 하셔도 무방합니다!

<br/>

# MobX 의 주요 개념들

1. Observable State (관찰 받고 있는 상태)

    MobX 에서는 정확히 **어떤 부분이 바뀌었는지** 알 수 있습니다.   
    그 값이, 원시적인 값이던, 객체이던, 배열 내부의 객체이던 객체의 키이던 간에 말이죠.  

2. Computed Value (연산된 값)

    기존의 상태값과 다른 연산된 값에 기반하여 만들어질 수 있는 값입니다.  
    주로 **성능 최적화**를 위하여 많이 사용됩니다.  

3. Reactions (반응)

    Reactions 는 Computed Value 와 비슷한데,    
    **Computed Value 의 경우는 우리가 특정 값을 연산해야 될 때 에만 처리**가 되는 반면에,  
    **Reactions 은, 값이 바뀜에 따라 해야 할 일을 정하는 것**을 의미합니다.   

4. Actions (액션; 행동)

    액션은, **상태에 변화를 일으키는것**을 말합니다.  
    만약에 Observable State 에 변화를 일으키는 코드를 호출한다? 이것은 하나의 액션입니다.   
    리덕스에서의 액션과 달리 따로 객체형태로 만들지는 않습니다.

<br>

# 리액트 없이 MobX 사용해보기
리덕스와 마찬가지로, MobX 는 리액트 종속적인 라이브러리가 아닙니다.   
그냥 따로 쓸 수도 있어요. **UI 프레임워크 / 라이브러리 없이** 쓰셔도 되고,   
Vue, Angular 등이랑 써도 전혀 무방합니다.  

```javascript
import { observable, reaction, computed, autorun } from 'mobx';
```

<br>

## **observable (관찰 받고 있는 상태)**

```javascript
import { observable, reaction, computed, autorun } from 'mobx';

// **** Observable State 만들기
const calculator = observable({
  a: 1,
  b: 2
});
```

Observable State 를 만들고나면 **MobX 가 이 객체를 "관찰 할 수" 있어서 변화가 일어나면 바로 탐지**해낼수있습니다.

<br>

## **reaction (반응)**

특정 값이 바뀔 때 어떤 작업을 하고 싶다면 reaction 함수를 사용합니다.

```javascript
import { observable, reaction, computed, autorun } from 'mobx';

// Observable State 만들기
const calculator = observable({
  a: 1,
  b: 2
});

// **** 특정 값이 바뀔 때 특정 작업 하기!
reaction(
  () => calculator.a,
  (value, reaction) => {
    console.log(`a 값이 ${value} 로 바뀌었네요!`);
  }
);

reaction(
  () => calculator.b,
  value => {
    console.log(`b 값이 ${value} 로 바뀌었네요!`);
  }
);

calculator.a = 10;
calculator.b = 20;
```

콘솔쪽을 보시면 다음과 같이 나타날 것입니다.

```javascript
a 값이 10 로 바뀌었네요! 
b 값이 20 로 바뀌었네요! 
```

<br>

## **computed (연산된 값)**

computed 함수는 연산된 값을 사용해야 할 때 사용됩니다.   
특징은, 이 값을 조회할 때 마다 특정 작업을 처리하는것이 아니라,   
이 값에서 **의존하는 값이 바뀔 때 미리 값을 계산해놓고 조회 할 때는 캐싱된 데이터를 사용**한다는 점 입니다.  

```javascript
mport { observable, reaction, computed, autorun } from 'mobx';

// Observable State 만들기
const calculator = observable({
  a: 1,
  b: 2
});

// **** 특정 값이 바뀔 때 특정 작업 하기!
reaction(
  () => calculator.a,
  (value, reaction) => {
    console.log(`a 값이 ${value} 로 바뀌었네요!`);
  }
);

reaction(
  () => calculator.b,
  value => {
    console.log(`b 값이 ${value} 로 바뀌었네요!`);
  }
);

// **** computed 로 특정 값 캐싱
const sum = computed(() => {
  console.log('계산중이예요!');
  return calculator.a + calculator.b;
});

sum.observe(() => calculator.a); // a 값을 주시
sum.observe(() => calculator.b); // b 값을 주시

calculator.a = 10;
calculator.b = 20;

//**** 여러번 조회해도 computed 안의 함수를 다시 호출하지 않지만..
console.log(sum.value);
console.log(sum.value);


// 내부의 값이 바뀌면 다시 호출 함
calculator.a = 20;
console.log(sum.value);
```

결과 값

```javascript
계산중이예요! 
a 값이 10 로 바뀌었네요! 
계산중이예요! 
b 값이 20 로 바뀌었네요! 
계산중이예요! 
30
30
a 값이 20 로 바뀌었네요! 
계산중이예요! 
40
```

<br/>

## **autorun**

autorun 은 reaction 이나 computed 의 observe 대신에 사용 될 수 있는데,   
autorun 으로 전달해주는 함수에서 사용되는 값이 있으면 자동으로 그 값을 주시하여
그 값이 바뀔 때 마다 함수가 주시되도록 해줍니다.   

여기서 만약에 computed 로 만든 값의 .get() 함수를 호출해주면, 하나하나 observe 해주지 않아도 됩니다.  

```javascript
import { observable, reaction, computed, autorun } from 'mobx';

// Observable State 만들기
const calculator = observable({
  a: 1,
  b: 2
});

// computed 로 특정 값 캐싱
const sum = computed(() => {
  console.log('계산중이예요!');
  return calculator.a + calculator.b;
});

// **** autorun 은 함수 내에서 조회하는 값을 자동으로 주시함
autorun(() => console.log(`a 값이 ${calculator.a} 로 바뀌었네요!`));
autorun(() => console.log(`b 값이 ${calculator.b} 로 바뀌었네요!`));
autorun(() => sum.get()); // su

calculator.a = 10;
calculator.b = 20;

// 여러번 조회해도 computed 안의 함수를 다시 호출하지 않지만..
console.log(sum.value);
console.log(sum.value);

calculator.a = 20;

// 내부의 값이 바뀌면 다시 호출 함
console.log(sum.value);
```

결과 값

```javascript
a 값이 1 로 바뀌었네요! 
b 값이 2 로 바뀌었네요! 
계산중이예요! 
a 값이 10 로 바뀌었네요! 
계산중이예요! 
b 값이 20 로 바뀌었네요! 
계산중이예요! 
30
30
a 값이 20 로 바뀌었네요! 
계산중이예요! 
40
```

<br/>


# class 문법을 사용해서 조금 더 깔끔하게

class 로 장바구니를 구현 후, decorate 함수를 통하여 MobX 를 적용해주겠습니다.

```javascript
import { decorate, observable, computed, autorun } from 'mobx';

class GS25 {
  basket = [];

  get total() {
    console.log('계산중입니다..!');
    // Reduce 함수로 배열 내부의 객체의 price 총합 계산
    // https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
    return this.basket.reduce((prev, curr) => prev + curr.price, 0);
  }

  select(name, price) {
    this.basket.push({ name, price });
  }
}

// decorate 를 통해서 각 값에 MobX 함수 적용
decorate(GS25, {
  basket: observable,
  total: computed,
});

const gs25 = new GS25();
autorun(() => gs25.total);
gs25.select('물', 800);
console.log(gs25.total);
gs25.select('물', 800);
console.log(gs25.total);
gs25.select('포카칩', 1500);
console.log(gs25.total);

```

결과 값

```javascript
계산중입니다..! 
계산중입니다..! 
800
계산중입니다..! 
1600
계산중입니다..! 
3100
```

<br/>

## **action**

상태에 변화를 일으키는것을 action 이라고 부른다고 언급했습니다. 
만약에, 이 변화를 일으키는 함수에 MobX 의 action 을 적용하면 무엇을 할 수 있는지 알아보겠습니다.

우선, 코드 상단에서 action 함수를 불러오고, decorate 쪽에 select 가 action 이라는 것을 명시해줄게요.

```javascript
// **** 액션 불러옴
import { decorate, observable, computed, autorun, action } from 'mobx';

class GS25 {
  basket = [];

  get total() {
    console.log('계산중입니다..!');
    // Reduce 함수로 배열 내부의 객체의 price 총합 계산
    // https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
    return this.basket.reduce((prev, curr) => prev + curr.price, 0);
  }

  select(name, price) {
    this.basket.push({ name, price });
  }
}

decorate(GS25, {
  basket: observable,
  total: computed,
  select: action // **** 액션 명시
});

const gs25 = new GS25();
autorun(() => gs25.total);
gs25.select('물', 800);
console.log(gs25.total);
gs25.select('물', 800);
console.log(gs25.total);
gs25.select('포카칩', 1500);
console.log(gs25.total);
```

action 을 사용함에 있어서의 이점은 나중에 개발자도구에서 변화의 세부 정보를 볼 수 있고,   
변화를 한꺼번에 일으켜서 변화가 일어날 때 마다 reaction 들이 나타나는것이 아니라,  
모든 액션이 끝나고 난 다음에서야 reaction 이 나타나게끔 해줄 수 있습니다.

액션을 한꺼번에 일으키는건, [transaction](https://mobx.js.org/refguide/transaction.html) 을 통해 할 수 있습니다.

우선 다음 예제의 콘솔 결과를 확인해보겠습니다.

```javascript
import {
  decorate,
  observable,
  computed,
  autorun,
  action,
  transaction // *** transaction 불러옴
} from 'mobx';

class GS25 {
  basket = [];

  get total() {
    console.log('계산중입니다..!');
    // Reduce 함수로 배열 내부의 객체의 price 총합 계산
    // https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
    return this.basket.reduce((prev, curr) => prev + curr.price, 0);
  }

  select(name, price) {
    this.basket.push({ name, price });
  }
}

decorate(GS25, {
  basket: observable,
  total: computed,
  select: action
});

const gs25 = new GS25();
autorun(() => gs25.total);
// *** 새 데이터 추가 될 때 알림
autorun(() => {
  if (gs25.basket.length > 0) {
    console.log(gs25.basket[gs25.basket.length - 1]);
  }
});

gs25.select('물', 800);
gs25.select('물', 800);
gs25.select('포카칩', 1500);

console.log(gs25.total);
```

결과 값

```javascript
계산중입니다..! 
계산중입니다..! 
Object {name: "물", price: 800}
계산중입니다..! 
Object {name: "물", price: 800}
계산중입니다..! 
Object {name: "포카칩", price: 1500}
3100
```

계산의 경우, 가장 처음 한번 호출이 되고, 데이터가 추가 될 때마다 계산이 되고 있습니다.  
한번 이걸 transaction 으로 감싸보겠습니다.

```javascript
import {
  decorate,
  observable,
  computed,
  autorun,
  action,
  transaction
} from 'mobx';

class GS25 {
  basket = [];

  get total() {
    console.log('계산중입니다..!');
    // Reduce 함수로 배열 내부의 객체의 price 총합 계산
    // https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
    return this.basket.reduce((prev, curr) => prev + curr.price, 0);
  }

  select(name, price) {
    this.basket.push({ name, price });
  }
}

decorate(GS25, {
  basket: observable,
  total: computed,
  select: action
});

const gs25 = new GS25();
autorun(() => gs25.total);
// 새 데이터 추가 될 때 알림
autorun(() => {
  if (gs25.basket.length > 0) {
    console.log(gs25.basket[gs25.basket.length - 1]);
  }
});

transaction(() => {
  gs25.select('물', 800);
  gs25.select('물', 800);
  gs25.select('포카칩', 1500);
})

console.log(gs25.total);
```

결과 값

```javascript
계산중입니다..! 
계산중입니다..! 
Object {name: "포카칩", price: 1500}
3100
```

transaction 을 통하여 계산 작업은 가장 처음 한번, 그리고 transaction 끝나고 한번 호출이 되었고, 새 데이터 추가 될 때마다 알리는 부분도 3개를 다 추가하고 나서야 딱 한번 콘솔에 마지막 객체가 나타났습니다.

액션을 사용하면, 이렇게 성능 개선도 이뤄낼 수 있고 나중에 MobX 개발자 도구를 사용하게 될 때 변화에 대한 더 자세한 정보를 알 수 있게 해줍니다.


# decorator 문법으로 더 편하게!

decorator 문법은 일종의, 자바스크립트 사투리 라고 생각하시면됩니다.   
정규 문법은 아니지만, babel 플러그인을 통하여 사용 할 수 있는 문법입니다.  
이 문법을 사용하면 decorate 함수가 더 이상 필요하지 않고,   
다음과 같이 작성 해 줄 수 있답니다.  

```javascript
import { observable, computed, autorun, action, transaction } from 'mobx';

class GS25 {
  @observable basket = [];

  @computed
  get total() {
    console.log('계산중입니다..!');
    // Reduce 함수로 배열 내부의 객체의 price 총합 계산
    // https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce
    return this.basket.reduce((prev, curr) => prev + curr.price, 0);
  }

  @action
  select(name, price) {
    this.basket.push({ name, price });
  }
}

const gs25 = new GS25();
autorun(() => gs25.total);
// 새 데이터 추가 될 때 알림
autorun(() => {
  if (gs25.basket.length > 0) {
    console.log(gs25.basket[gs25.basket.length - 1]);
  }
});

transaction(() => {
  gs25.select('물', 800);
  gs25.select('물', 800);
  gs25.select('포카칩', 1500);
});

console.log(gs25.total);
```

결과는 아까와 같습니다.

MobX 를 사용하는 대부분의 예제는 이 Decorator 를 사용합니다.  
아무래도, 사용하는 편이 훨씬 편하긴 하지만 없이도 사용 하는 것에는 지장이 안갑니다.  