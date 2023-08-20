---
layout: post
title: "[React] 5장. state와 stateless"
description: " "
date: 2021-01-15
tags: [React]
comments: true
share: true
---


## React.js 기초 다지기

#### 목차

1. 리액트란?

2. JSX란?

3. 컴포넌트란?

4. props와 state

5. **state와 stateless**

6. style 적용하기

7. life cycle

    

------

## 5. state와 stateless

컴포넌트 클래스에는 state와 stateless한 상태가 있음.
- **state**한 클래스의 경우 : 스스로 초기 state를 설정해줘야 하므로 생성자 메소드를 필요로 한다.
- **stateless**한 클래스의 경우 : 부모든 뭐든 state를 넘겨주는 걸 받아서 쓰므로 생성자로 설정해 줄 필요가 없다.



___

### 자식 컴포넌트가 부모 컴포넌트의 state update하기

1. 부모 컴포넌트 클래스는 this.setState()를 호출하는 이벤트 핸들러 메소드를 가지고 있어야 한다.
2. 새로 정의된 메소드를 인스턴스에 전달하기 위해 생성자에
   **`this.handleClick = this.handleClick.bind(this);`**
   이런 식으로 추가해줘야 한다.
3. 부모 컴포넌트가 자신의 state를 변경하는 이벤트 핸들러 메소드를 자식 컴포넌트에게 props로 넘겨준다.
4. 자식 컴포넌트는 props로 넘겨 받은 메소드를 갖다 쓰면 된다.



___

### 자식 컴포넌트가 형제 컴포넌트의 state update하기

- 한 stateless한 컴포넌트는 정보를 보여주고, 다른 stateless한 컴포넌트는 정보를 업데이트한다.
- 방법은 위와 유사한데, 새로운 자식 컴포넌트가 부모 컴포넌트의 state를 가져와서 화면에 뿌려준다는 점이 다름.

1. state한 컴포넌트 클래스(부모 컴포넌트)는 this.setState를 호출하는 메소드를 가진다.
2. state한 컴포넌트는 stateless한 컴포넌트(자식 컴포넌트A)에게 그 메소드를 넘긴다.
3. 자식 컴포넌트A는 부모로부터 받아온 메소드를 호출하는 함수를 하나 정의하고, 이벤트 오브젝트를 인자로 넘긴다.
4. stateless한 컴포넌트 클래스는 방금 정의한 함수를 이벤트 핸들러로 쓴다.
5. 이벤트가 감지되면 부모 컴포넌트의 state가 업데이트된다.
6. 부모 컴포넌트는 다른 자식 컴포넌트B에게 state를 넘긴다.
7. 다른 자식 컴포넌트B는 그 state를 받아서 render한다.
8. 부모 컴포넌트는 다른 자식 컴포넌트B를 포함해 화면에 뿌려준다.



***

### 리액트의 프로그래밍 패턴1

* 컨테이너 컴포넌트를 presentational 컴포넌트에서 떼어 낸다.
  - 컴포넌트가 state를 가져야 하고, props로부터 계산되어야 하고... 복잡한 로직을 가져야 한다면 컴포넌트는 HTML 렌더링까지 시키기 좀 미안하다.
  - presentational 컴포넌트는 보여주는게 목적이니까 render() 함수만 멤버로 가지게 된다.
    > 그럴거면 그냥 **render할 JSX만 return하는 함수를 담은 변수로 만들어서 export** 해버리자!

    - **_stateless functional Component_**
    - 함수의 인자로 (props)를 줄 수도 있음. return하기 전에 해당 props에 대한 처리를 해주면 된다.

