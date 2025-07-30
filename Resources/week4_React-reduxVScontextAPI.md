# Redux란? 🔮

리덕스는 상태의 중앙 관리를 위한 **상태 관리 도구**입니다. React뿐만 아니라 Angular, Vue에서도 사용할 수 있습니다. 한 마디로, 리덕스는 '전역 상태'를 생성하고 관리하기 위한 라이브러리라고 볼 수 있습니다.

리덕스는 크게 전역 상태를 보관하는 **저장소**, 상태 저장소에 접근을 위한 **리듀서**, 리듀서에 행동을 지시하는 **액션**, 저장소에 보관된 상태를 가져오는 **서브스크립션**으로 나뉘어져 있습니다.  
![](https://velog.velcdn.com/images%2Fcada%2Fpost%2F8356b3e0-6edd-4512-8dd8-d77c2cecc219%2F1_Fo8IkcnT_uktIeCq-Wo9xg.png)

## store

전역 상태를 저장합니다. 자바스크립트 객체 형태로 저장되어 있으며 리듀서를 통하지 않고 접근할 수 없습니다.

하나의 어플리케이션에 하나의 저장소만 존재해야합니다. 리액트에서는 주로 `index.js`에 정의합니다. 저장소가 하나이기 때문에 리듀서 또한 하나로 존재해야합니다.

또, immutable하다는 특성을 가지고 있습니다.

```javascript
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

import { createStore } from 'redux';
import { Provider } from 'react-redux';

const store = createStore(/*your root reducer*/);

ReactDOM.render(
  <Provider store={store}>
  	<App />
  </Provider>,
  document.getElementById('root')
);
```

## reducer

저장소에 유일하게 접근할 수 있는 유일한 객체입니다. 들어오는 **action**에 따라 행동합니다. 리듀서 함수 내에서 반환되는 값이 상태 저장소에 저장됩니다. 단, 상태가 추가되는 것이 아닌 덮어씌워지게 되므로 전체 상태를 복사하여 상태를 갱신한 후에 반환해야 합니다.

즉, 저장소에 {count: 1, price: 100} 이 저장되어 있는 상태에서 리듀서가 {count: 2} 를 반환하면 저장소에는 {count: 2} 가 저장됩니다.

```javascript
// 기본 상태값을 지정할 수 있습니다. (initState)
const rootReducer = (state = initState, action) => {
  if (action.type === 'DELETE_POST') {
    const newPosts = state.posts.filter((post) => post.id !== action.id);
    return {
      ...state,
      posts: newPosts,
    };
  }
  return state;
};
```

## action

액션은 리듀서에게 보내지는 저장소에 대한 행동입니다. 리듀서는 이 행동에 따라 저장소에 접근하여 정해진 동작을 하게 됩니다. 즉, **트리거(trigger)** 역할이라고 볼 수 있습니다.

```javascript
export const deletePost = (id) => {
  return {
    type: 'DELETE_POST',
    id: id,
  };
};
```

## subscription

서브스크립션은 저장소에 저장된 전역 상태를 가져옵니다. 따라서, 어느 컴포넌트에서도 저장소에서 상태값을 얻을 수 있습니다.

이를 통해 얻은 액션과 전역상태를 통해 컴포넌트는 원하는 동작을 할 수 있게 됩니다.

```javascript
// Post.js
import { connect } from 'react-redux';

const mapStateToProps = (state, ownProps) => {
  return {
    posts: state.posts,
  };
};

const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    deletePost: (id) => {
      dispatch(deletePost(id));
    },
  };
};

export default connect(mapStateToProps, mapDispatchToProps)(Post);
```

위 코드에서, HOC를 통해 전역상태와 액션을 위한 Dispatch를 컴포넌트에 전달합니다.  
이 컴포넌트에서는 `props`를 통해 전역상태와 Dispatch를 사용할 수 있습니다.

  

# Context API란? 💎

Context API는 Redux와 마찬가지로 상태의 중앙 관리를 위한 **상태 관리 도구**입니다. Redux와 다르게 React에서만 사용할 수 있습니다. 리덕스와 다르게 여러 저장소가 존재할 수 있습니다.

Context API는 크게 전역 상태가 저장되는 **context**, 전역 상태를 제공하는 **Provider**, 그리고 전역 상태를 받아 사용하는 **Consumer**로 나뉘어져 있습니다.

## context

context는 컴포넌트가 Provide한 상태가 저장됩니다. Consumer는 이 context를 통해 전역 상태에 접근할 수 있습니다. 여러 context가 존재할 수 있습니다. 단, 하나의 context만 존재한다면 성능상 이슈가 있을 수 있으니 유의할 필요가 있습니다.

context 안에는 Provider와 Consumer가 정의되어 있으며 다른 컴포넌트에서는 이것들을 이용해 상태에 접근합니다.

```javascript
// ./contexts/root.js
import React from 'react'

export default React.createContext({}) // 함수의 인자에는 상태의 초기값이 들어갑니다.
```

## Provider

context에 상태를 제공합니다. 즉, 다른 컴포넌트가 해당 상태에 접근하여 사용할 수 있습니다.

제공된 상태에 접근하기 위해서는 Provider의 하위 컴포넌트에 포함되어야 합니다.  
따라서, 모든 컴포넌트에서 접근해야하는 상태를 제공하기 위해서는 루트 컴포넌트 `index.js` 혹은 `app.js`에서 Provider를 정의해야 합니다.

```javascript
import ShopContext from './path/to/shop-context'; // React.createContext() 객체

class App extends Component {
  render() {
    return (
      <ShopContext.Provider value={{
          products: [],
          cart: []
        }
      }>
        {/* 이곳 Provider 사이에 있는 컴포넌트는 전역 상태에 접근할 수 있습니다. */}
      </ShopContext.Provider />
    );
  }
}
```

## consumer

consumer는 제공된 상태에 접근하는 방법 중 하나입니다. context는 Consumer 사이에 있는 처음의 객체를 context에 인자로 전달하기 때문에 바로 JSX를 작성하는 것이 아닌 빈 객체를 작성하고 나서 JSX를 작성해야 합니다.

```javascript
import ShopContext from '../context/shop-context' // React.createContext() 객체

class ProductsPage extends Component {
  render() {
    return (
      <ShopContext.Consumer>  { }
        {context => (
          <React.Fragment>
            <MainNavigation
              cartItemNumber={context.cart.reduce((count, curItem) => {
                return count + curItem.quantity
              }, 0)}
            />
            <main className="products">...</main>
          </React.Fragment>
        )}
      </ShopContext.Consumer>
    )
  }
}

export default ProductsPage
```

## contextType

클래스 컴포넌트에서 static 타입을 이용하여 전역 상태에 접근할 수 있습니다. Consumer와 다르게 함수형 컴포넌트에서는 사용할 수 없습니다. 하지만 React Hooks를 이용한다면 함수형 컴포넌트에서도 사용할 순 있습니다.

```javascript
import ShopContext from '../context/shop-context'

class CartPage extends Component {
  static contextType = ShopContext

  componentDidMount() {
    // 별도의 복잡한 코드 작성 없이 다른 컴포넌트에서 해당 상태에 접근할 수 있습니다.
    console.log(this.context)
  }

  render() {
    return (
      <React.Fragment>
        <MainNavigation
          cartItemNumber={this.context.cart.reduce((count, curItem) => {
            return count + curItem.quantity
          }, 0)}
        />
        <main className="cart">...</main>
      </React.Fragment>
    )
  }
}

export default CartPage
```

Context API에서는 상태의 갱신을 위해서 value를 Provide한 것 처럼 상태의 갱신을 위한 함수도 value와 함께 Provide하여 하위 컴포넌트에서 사용해야 합니다. 하위 컴포넌트에 Props에 함수를 전달한 것처럼 말이죠.

  

# Redux와 Context API의 차이 ✔️

위의 설명만 보면은 Redux와 Context API는 사실상 차이가 거의 없어보입니다. 둘 다 전역 상태 관리를 위한 도구라는 공통점을 가지고 있기 때문이죠. 사실 Redux 또한 Context API를 가지고 만든 라이브러리입니다. 전역 상태 관리 측면에서는 차이점이 거의 없다고 봐도 무방하다는 의미입니다. _(Context API는 high-frequency updates에 좋지 않은 성능을 보이지만 Redux는 그렇지 않습니다)_

하지만 Redux는 Context API와 다르게 전역 상태 관리외에 다양한 기능을 제공합니다.

![](https://velog.velcdn.com/images%2Fcada%2Fpost%2F2fe54f52-a384-444a-88ad-05fd2d10028c%2Fdas.PNG)  
Dan Abrarnov의 Medium 게시글을 발췌한 부분입니다.  
위 항목들은 모두 Redux가 Context API에 비해 가지는 강점입니다. `redux-saga`, `redux-thunk`, `redux-devtools` 등 다양한 추가 라이브러리를 통해 우리가 조금 더 상태 관리를 수월하게하고 긴밀하고 정확한 코딩을 할 수 있도록 합니다. 여러 라이브러리가 모여 Redux라는 하나의 프레임워크가 되어 개발자에게 큰 도움을 주고 있는 것으로 볼 수 있습니다. 하지만 Context API는 이런 부가적인 기능을 제공하지 않아 다른 라이브러리를 통해 구현해야합니다.

Redux가 많은 기능을 제공하지만 Context API에 비해 작성해야하는 코드도 많고 복잡하기 때문에 이런 부가 기능이 필요하지 않다면 Redux를 사용하지 않아도 됩니다.

  

# 결론 📃

- 오직 전역 상태 관리를 위한다면 Context API를 사용하라.
- 상태 관리 외에 여러 기능이 필요하다면 Redux 를 사용하라.
- high-frequency한 어플리케이션의 경우 Context API를 사용하면 성능상 이슈가 있을 수 있다.


https://velog.io/@cada/React-Redux-vs-Context-API