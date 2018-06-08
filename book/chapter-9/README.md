# Redux

[Redux](https://redux.js.org/) เป็น Library ที่ทำตัวเป็นตัวเก็บ state และช่วยจัดการ data ภายใน application เปิดตัวครั้งแรกในงาน ReactEurope conference ([video](https://www.youtube.com/watch?v=xsSnOQynTHs)) ในปี 2015 โดย [Dan Abramov](https://twitter.com/dan_abramov) มีลักษณะการทำงานและการออกแบบคล้าย [Flux architecture](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-8/README.md#flux-architecture-and-its-main-characteristics)

[Redux](https://redux.js.org/) is a library that acts as a state container and helps managing your application data flow. It was introduced back in 2015 at ReactEurope conference ([video](https://www.youtube.com/watch?v=xsSnOQynTHs)) by [Dan Abramov](https://twitter.com/dan_abramov). It is similar to [Flux architecture](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-8/README.md#flux-architecture-and-its-main-characteristics) and has a lot in common with it. In this section we will create a small counter app using Redux alongside React.

<span class="new-page"></span>

## สถาปัตยกรรม Redux และลักษณะสำคัญ
## Redux architecture and its main characteristics

![Redux architecture](./redux-architecture.jpg)

เช่นเดียวกับ Flux Redux มี view components (React) ที่คอย dispatch action โดยที่ action เดียวกันสามารถถูก dispatch มาจากส่วนไหนของระบบก็ได้ ยกตัวอย่างเช่น การเรียก bootstrap เป็นต้น action ที่ถูก dispatch จะถูกส่งตรงไปยัง store ซึ่งมีแค่ตัวเดียวเท่านั้นใน Redux ซึ่งสิ่งนี้เป็นสิ่งที่ Redux ไม่เหมือนกับ Flux ส่วนที่จะตัดสินใจว่า data ของเราจะเปลี่ยนไปอย่างไรนั้นขึ้นอยู่กับ reducers ที่เป็น pure functions เมื่อไรก็ตามที่ store ได้รับ action reducers จะทำการรับ current state และ action ที่ถูกส่งเข้ามา เพื่อคำนวนและสร้าง state ถัดไป โดยอิงหลัก immutable store จะรับช่วงต่อและเปลี่ยนค่า state ภายใน store สุดท้าย React component ที่ดึง data มาจาก store ก็จะถูก re-render

Concept ของ Redux ค่อนข้างตรงไปตรงมาโดยยึดหลัก [one-direction data flow](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-7/README.md) เรามาเริ่มพูดถึงเรื่องนี้และแนะนำส่วนที่ช่วยสนับสนุน Redux pattern กันดีกว่า

Similarly to [Flux](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-8/README.md) architecture we have the view components (React) dispatching an action. Same action may be dispatched by another part of our system. Like a bootstrap logic for example. This action is dispatched not to a central hub but directly to the store. We are saying "store" not "stores" because there is only one in Redux. That is one of the big differences between Flux and Redux. The logic that decided how our data changes lives in pure functions called reducers. Once the store receives an action it asks the reducers about the new version of the state by sending the current state and the given action. Then in immutable fashion the reducer needs to return the new state. The store continues from there and updates its internal state. As a final step, the wired to the store React component gets re-rendered.

The concept is pretty linear and again follows the [one-direction data flow](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-7/README.md). Let's talk about all these pieces and introduce a couple of new terms that support the work of the Redux pattern.

### Actions

การจำแนกชนิดของ action ใน Redux จะเหมือนกับ Flux คือแบ่งได้ด้วย property ของ object ที่ชื่อว่า `type` โดยที่ data อื่นๆที่ส่งมากับ object จะใช้กับ logic ของ application เท่านั้น จะไม่เกี่ยวข้องกับ Redux pattern ยกตัวอย่างเช่น

The typical action in Redux (same as Flux) is just an object with a `type` property. Everything else in that object is considered a context specific data and it is not related to the pattern but to your application logic. For example:

```js
const CHANGE_VISIBILITY = 'CHANGE_VISIBILITY';
const action = {
  type: CHANGE_VISIBILITY,
  visible: false
}
```
ถือว่าเป็นตัวอย่างที่ดีในการสร้าง action type เป็น constants `CHANGE_VISIBILITY` ซึ่งยังมี tools หรือ libraries หลายๆอย่างที่รองรับ Redux ที่มีวีธีเรียกใช้ที่สะดวกโดยการส่ง action type อย่างเดียว

It is a good practice that we create constants like `CHANGE_VISIBILITY` for our action types. It happens that there are lots of tools/libraries that support Redux and solve different problems which do require the type of the action only. So it is just a convenient way to transfer this information.

ส่วนของ property `visible` เป็น metadata ที่เราได้กล่าวถึงแล้วว่า ไม่ได้ถูกใช้ใน Redux เป็นเพียงแค่ข้อมูลที่ใช้ใน application เท่านั้น

The `visible` property is the meta data that we mentioned above. It has nothing to do with Redux. It means something in the context of the application.

ทุกๆครั้งที่เราต้องการ dispatch method เราต้องใช้ object ซึ่งมันเป็นการยุ่งยากถ้าจะต้องมาเขียนมันซ้ำแล้วซ้ำเล่า ซึ่งเป็นที่มาของ *action creators* ที่เป็น function ที่ return ค่า object และจะรับหรือไม่รับ argument ที่เกี่ยวข้องกับ action นั้นเพิ่มก็ได้ ยกตัวอย่างเช่น action creator ของ action ด้านบน จะมีหน้าตาตามด้านล่าง

Every time when we want to dispatch a method we have to use such objects. However, it becomes too noisy to write them over and over again. That is why there is the concept of *action creators*. An action creator is a function that returns an object and may or may not accept an argument which directly relates to the action properties. For example the action creator for the above action looks like this:

```js
const changeVisibility = visible => ({
  type: CHANGE_VISIBILITY,
  visible
});

changeVisibility(false);
// { type: CHANGE_VISIBILITY, visible: false }
```

จะสังเกตได้ว่าเราทำการส่งค่าของ `visble` ผ่าน argrument ทำให้เราไม่ต้องจดจำค่าจริงของ action type นั้น ซึ่งการใช้ตัวช่วยพวกนี้จะทำให้ code ของเราสั้นและง่ายต่อการอ่าน

Notice that we pass the value of the `visible` as an argument and we don't have to remember (or import) the exact type of the action. Using such helpers makes the code compact and easy to read.

### Store

Redux ได้เตรียมตัวช่วยอย่าง `createStore` ไว้สำหรับการสร้าง store โดยมี signature ดังนี้

Redux provides a helper `createStore` for creating a store. Its signature is as follows:

```js
import { createStore } from 'redux';

createStore([reducer], [initial state], [enhancer]);
```

เราได้กล่าวถึงไว้แล้วว่า reducer เป็น function ที่รับ current state และ action และ return ค่าเป็น state ใหม่ ต่อมา argrument ที่สองคือ state เริ่มต้น ซึ่งมีประโยชน์สำหรับในการใช้เป็นการกำหนดค่า state เมื่อ application เริ่มทำงาน ซึ่ง feature นี้เป็นหนึ่งใน process ของการทำ server-side rendering หรือ persistent experience ส่วน argument ที่สามคือ enchancer เป็นส่วนไว้สำหรับต่อกับ third party API หรือ plug กับ function ที่ไม่ได้มีใน Redux ยกตัวอย่างเช่น วิธีการรับมือกับ async processes

We already mentioned that the reducer is a function that accepts the current state and action and returns the new state. More about that in a bit. The second argument is the initial state of the store. This comes as a handy instrument to initialize our application with data that we already have. This feature is the essence of processes like server-side rendering or persistent experience. The third parameter, enhancer, provides an API for extending Redux with third party middlewares and basically plug some functionally which is not baked-in. Like for example an instrument for handling async processes.

store ที่สร้างขึ้นมาประกอบไปด้วย 4 method คือ `getState`, `dispatch`, `subscribe` และ `replaceReducer` ซึ่งตัวที่สำคัญที่สุดคือ `dispatch`

Once created the store has four methods - `getState`, `dispatch`, `subscribe` and `replaceReducer`. Probably the most important one is `dispatch`:

```js
store.dispatch(changeVisibility(false));
```

จากด้านบนเป็นวิธีที่เราเรียกใช้ action creators โดยเราจะส่งผลรับที่ได้จาก action creators ซึ่งก็คือ action object ไปยัง `dispatch` method ซึ่งจะถูกกระจายต่อไปยัง reducer แต่ละตัวที่อยู่ใน application

That is the place where we use our action creators. We pass the result of them or in other words action objects to this `dispatch` method. It then gets spread to the reducers in our application.

โดยทั่วไปแล้วใน React application เราจะไม่ค่อยได้ใช้ `getState` และ `subscribe` ตรงๆ เพราะเรามีตัวช่วย (เราจะอธิบายในส่วนต่อไป) ในการที่จะเชื่อมต่อ component ของเราเข้ากับ store และ `subscribe` อย่างมีประสิทธิภาพเมื่อมีการเปลี่ยนแปลง ในส่วนของ subscription เรายังได้รับ current state ทำให้ไม่ต้อง call `getState` เองอีกด้วย ส่วน `replaceReducer` เป็น API ที่ค่อนข้างจะซับซ้อน ใช้สำหรับเปลี่ยน reducer ที่กำลังถูก store ใช้งานอยู่ โดยส่วนตัวแล้วยังไม่มีโอกาสได้ใช้เลย

In the typical React application we usually don't use `getState` and `subscribe` directly because there is a helper (we will see it in the next sections) that wires our components with the store and effectively `subscribe`s for changes. As part of this subscription we also receive the current state so we don't have to call `getState` ourself. `replaceReducer` is kind of an advanced API and it swaps the reducer currently used by the store. I personally never used this method.

### Reducer

reducer เป็น function ที่เรียกได้ว่า *สวยงามที่สุด* ภายใน Redux ถึงแม้ก่อนหน้านี้ ผมจะเริ่มสนใจในการเขียน pure fuction และ immutability แต่ Redux บังคับให้ผมต้องเขียน เพราะมีลักษณะสำคัญ 2 อย่างของ reducer ที่มีความสำคัญมาก หากขาดไปแล้ว pattern จะไม่มีทางเกิดขึ้น

The reducer function is probably the most *beautiful* part of Redux. Even before that I was interested in writing pure functions with an immutability in mind but Redux forced me to do it. There are two characteristics of the reducer that are quite important and without them we basically have a broken pattern.

(1) ต้องเป็น pure function เท่านั้น - หมายถึงว่า function ควรจะ return ค่าเดียวกันทุกครั้งหากมี input ที่เหมือนกัน

(1) It must be a pure function - it means that the function should return the exact same output every time when the same input is given.

(2) ควรจะไม่มี side effects - ของจำพวก global variable, การสร้าง async call หรือ การรอ promise เพื่อที่จะ resolve ไม่ควรมี


(2) It should have no side effects - stuff like accessing a global variable, making an async call or waiting for a promise to resolve have no place in here.

นี่คือตัวอย่างง่ายๆของ counter reducer

Here is a simple counter reducer:

```js
const counterReducer = function (state, action) {
  if (action.type === ADD) {
    return { value: state.value + 1 };
  } else if (action.type === SUBTRACT) {
    return { value: state.value - 1 };
  }
  return { value: 0 };
};
```

จะเห็นว่าไม่มี side effects และเรา return oject ตัวใหม่ทุกครั้ง เราเพิ่มค่าให้กับ value โดยอิงจาก previous state และ action type ที่ถูกส่งเข้ามา

There are no side effects and we return a brand new object every time. We accumulate the new value based on the previous state and the incoming action type. 


### การเชื่อมต่อกับ React components
### Wiring to React components

ถ้าเราพูดถึง Redux ที่ใช้ใน React แล้วมักจะหมายถึง ซึ่งประกอบด้วยสองอย่างที่ช่วยเชื่อมต่อ Redux กับ components ของเรา

If we talk about Redux in the context of React we almost always mean [react-redux](https://github.com/reactjs/react-redux) module. It provides two things that help connecting Redux to our components.

(1) `<Provider>` component - เป็น component ที่รับ store เข้ามาเพื่อทำให้ children node ใน React tree สามารถ access ได้โดยผ่าน React's content API ตัวอย่างเช่น

(1) `<Provider>` component - it's a component that accepts our store and makes it available for the children down the React tree via the React's context API. For example:

```js
<Provider store={ myStore }>
  <MyApp />
</Provider>
```

โดยปกติแล้วเราจะประกาศใช้แค่ที่เดียวใน app

We usually have a single place in our app where we use it.

(2) `connect` function - เป็น function ที่ใช้ทำ subcribe เพื่ออัพเดท store และ re-render component โดย function จะสร้างเป็น [higher-order component](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-4/README.md#higher-order-component) ออกมา โดย function มี signature ดังนี้

(2) `connect` function - it is a function that does the subscribing for updates in the store and re-renders our component. It implements a [higher-order component](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-4/README.md#higher-order-component). Here is its signature:

```
connect(
  [mapStateToProps],
  [mapDispatchToProps],
  [mergeProps],
  [options]
)
```

`mapStateToProps` เป็น function ที่รับ currenct state และ return เป็น set ของ key-value (object) ที่จะถูกส่งไปยัง React componnet ในรูปของ props, ยกตัวอย่างเช่น

`mapStateToProps` parameter is a function that accepts the current state and must return a set of key-value pairs (an object) that are getting send as props to our React component. For example:

```js
const mapStateToProps = state => ({
  visible: state.visible
});
```

`mapDispatchToProps` คล้ายกับ mapStateToProps แต่จะรับ `dispatch` function แทน `state` ซึ่งจุดนี้จะเป็นที่ๆเราจะประกาศ prop สำหรับ dispatch action

`mapDispatchToProps` is a similar one but instead of the `state` receives a `dispatch` function. Here is the place where we can define a prop for dispatching actions.

```js
const mapDispatchToProps = dispatch => ({
  changeVisibility: value => dispatch(changeVisibility(value))
});
```

`mergeProps` เป็นที่รวม และ เข้าด้วยกัน เพื่อให้โอกาสเปลี่ยนแปลงค่า props ก่อนจะส่งไปยัง component จากตัวอย่างด้านบน ถ้าเราต้องการที่จะทำ action สองอย่าง เราสามารถรวมมันเข้าด้วยกันเป็น props เดียวแล้วส่งไปยัง React ได้ `option` รับ setting ไว้สำหรับควบคุมการทำงานของ connect function

`mergeProps` combines both `mapStateToProps` and `mapDispatchToProps` and the props send to the component and gives us the opportunity to accumulate better props. Like for example if we need to fire two actions we may combine them to a single prop and send that to React. `options` accepts couple of settings that control how the connection works.

<br />

## Simple counter app using Redux

เรามาเริ่มสร้าง counter app แบบง่ายๆ ที่ใช้ APIs จากด้านบนกัน

Let's create a simple counter app that uses all the APIs above.

![Redux counter app example](./redux-counter-app.png)

ปุ่ม "Add" และ "Subtract" จะเป็นตัวเปลี่ยนแปลงค่าที่อยู่ใน store ของเรา "Visible" และ "Hidden" จะเป็นตัวควบคุมการแสดงค่า 

The "Add" and "Subtract" buttons will simply change a value in our store. "Visible" and "Hidden" will control its visibility.

### Modeling the actions

สำหรับผมแล้ว ทุก Redux feature จะเริ่มต้นด้วยการขึ้น model ของ action types และประกาศสิ่งที่จะเก็บใน state ในกรณีนี้เรามี 3 operation คือ adding, subtracting และ Visible/Hidden ดังนั้นเราจะมาเริ่มจาก: 

For me, every Redux feature starts with modeling the action types and defining what state we want to keep. In our case we have three operations going on - adding, subtracting and managing visibility. So we will go with the following:

```js
const ADD = 'ADD';
const SUBTRACT = 'SUBTRACT';
const CHANGE_VISIBILITY = 'CHANGE_VISIBILITY';

const add = () => ({ type: ADD });
const subtract = () => ({ type: SUBTRACT });
const changeVisibility = visible => ({
  type: CHANGE_VISIBILITY,
  visible
});
```
### Store กับ reducers
### Store and its reducers

ยังมีบางอย่างที่เรายังไม่ได้พูดถึงตอนที่เราอธิบายเรื่อง store กับ reducer, โดยปกติแล้วเราจะมี reducer มากกว่าหนึ่งตัว เพราะเราต้องการที่จะแยกจัดการหลายๆอย่าง เรามี store อยู่ตัวเดียวอยู่แล้วและตามทฤษฏีแล้ว state ก็จะมีแค่ตัวเดียวเหมือนกัน โดยส่วนใหญ่ application ที่รันบน production จะมี state ที่ถูกแบ่งเป็นส่วนๆ โดยแต่ละส่วนจะแสดงให้เห็นถึงแต่ละส่วนของระบบ ในตัวอย่างเรามีส่วนของ counting และ visibility ซึ่งเราสามารถสร้าง state ได้ตามนั้นเลย

There is something that we didn't talk about while explaining the store and reducers. We usually have more then one reducer because we want to manage multiple things. The store is one though and we in theory have only one state object. What happens in most of the apps running in production is that the application state is a composition of slices. Every slice represents a part of our system. In this very small example we have counting and visibility slices. So our initial state looks like that:

```js
const initialState = {
  counter: {
    value: 0
  },
  visible: true
};
```

เราต้องสร้าง reducer แยกกันสำหรับแต่ละส่วนซึ่งทำให้ code ของเรามีความยืดหยุ่นและอ่านง่ายมากขึ้น ลองคิดดูว่าถ้าเรามี app ที่มีขนาดใหญ่ที่มีการแบ่ง state มากกว่า 10 ส่วน มันคงเป็นการยากที่เราจะต้องจัดการมันอยู่บน function เดียว

We must define separate reducers for both parts. This is to introduce some flexibility and to improve the readability of our code. Imagine if we have a giant app with ten or more state slices and we keep working within a single function. It will be too difficult to manage.

Redux มาพร้อมกับ function `combineReducers` ที่ช่วยให้เราสามารถ assign reducer ในลงใน state แบบเจาะจง

Redux comes with a helper that allows us to target a specific part of the state and assign a reducer to it. It is called `combineReducers`:

```js
import { createStore, combineReducers } from 'redux';

const rootReducer = combineReducers({
  counter: function A() { ... },
  visible: function B() { ... }
});
const store = createStore(rootReducer);
```

function `A` จะรับเฉพาะส่วน `counter` จาก state และต้อง return ค่าเฉพาะส่วนนั้น เช่นเดียวกับ `B` ที่จะรับ boolean (ค่าของ `visible`) และต้อง return boolean เท่านั้น

Function `A` receives only the `counter` slice as a state and needs to return only that part. Same for `B`. Accepts a boolean (the value of `visible`) and must return a boolean.

reducer สำหรับส่วน counter ควรจะต้องรับ action `ADD` และ `SUBTRACT` มาเพื่อคำนวนหาค่า `counter` state ใหม่

The reducer for our counter slice should take into account both actions `ADD` and `SUBTRACT` and based on them calculates the new `counter` state.

```js
const counterReducer = function (state, action) {
  if (action.type === ADD) {
    return { value: state.value + 1 };
  } else if (action.type === SUBTRACT) {
    return { value: state.value - 1 };
  }
  return state || { value: 0 };
};
```

reducer ทุกตัวจะถูกเรียกอย่างน้อยหนึ่งครั้งตอนเริ่มสร้าง store โดยค่าเริ่มต้นของ `state` จะเป็น `undefined` และ `action` จะมีค่าเป็น `{ type: "@@redux/INIT"}` ซึ่งในกรณีนี้ reducer ของเราควรจะ return ค่าเริ่มต้นเป็น `{ value: 0 }`

Every reducer is fired at least once when the store is initialized. In that very first run the `state` is `undefined` and the `action` is `{ type: "@@redux/INIT"}`. In this case our reducer should return the initial value of our data - `{ value: 0 }`.

reducer สำหรับ

The reducer for the visibility is pretty similar except that it waits for `CHANGE_VISIBILITY` action:

```js
const visibilityReducer = function (state, action) {
  if (action.type === CHANGE_VISIBILITY) {
    return action.visible;
  }
  return true;
};
```

And at the end we have to pass both reducers to `combineReducers` so we create our `rootReducer`.

```js
const rootReducer = combineReducers({
  counter: counterReducer,
  visible: visibilityReducer
});
```

### Selectors

ก่อนจะเริ่มในส่วนถัดไป React components ที่เราได้กล่าวไว้ในส่วน concept ของ *selector* จาก section ที่แล้ว เรารู้แล้วว่า state ของเราถูกแบ่งออกเป็นหลายๆส่วน เราได้ให้ reducer จัดการเกี่ยวกับ update data แต่เมื่อไหร่ที่มีการเรียก data เรายังคงมี state object ตัวเดียวอยู่ ซึ่งส่วนนี้ selector จะเป็นตัวช่วยจัดการ โดย selector จะเป็น function ที่รับ state oject และเลือก return เฉพาะ data ที่เราต้องการ ยกตัวอย่างเช่น app เล็กๆของเราต้องการแค่ selector สองตัวนี้

Before moving to the React components we have to mention the concept of a *selector*. From the previous section we know that our state is usually divided into different parts. We have dedicated reducers to update the data but when it comes to fetching it we still have a single object. Here is the place where the selectors come in handy. The selector is a function that accepts the whole state object and extracts only the information that we need. For example in our small app we need two of those:

```js
const getCounterValue = state => state.counter.value;
const getVisibility = state => state.visible;
```

counter app เล็กเกินกว่าที่จะเห็นประสิทธิภาพจริงๆของการเขียนตัวช่วยพวกนี้ แต่ใน project ใหญ่ๆจะต่างกันกันมาก ไม่ใช่แค่เพียงการที่เขียน code น้อยลงหรืออ่านง่าย เพราะ selector อาจมาพร้อมกับส่วนอื่นๆที่อาจจะมี logic อยู่เนื่องจาก selector สามารถเข้าถึง state ได้ทั้งหมดทำให้สามารถใส่ business logic เพื่อตอบคำถามอย่างเช่น "user มีสิทธิ์ที่จะทำ X ในขณะที่อยู่หน้า Y ได้หรือไม่" ซึ่งสามารถจัดการได้ใน selector เดียว

A counter app is too small to see the real power of writing such helpers. However, in a big project is quite different. And it is not just about saving a few lines of code. Neither is about readability. Selectors come with these stuff but they are also contextual and may contain logic. Since they have access to the whole state they are able to answer business logic related questions. Like for example "Is the user authorize to do X while being on page Y". This may be done in a single selector.

### React components

Let's first deal with the UI that manages the visibility of the counter.

```js
function Visibility({ changeVisibility }) {
  return (
    <div>
      <button onClick={ () => changeVisibility(true) }>
        Visible
      </button>
      <button onClick={ () => changeVisibility(false) }>
        Hidden
      </button>
    </div>
  );
}

const VisibilityConnected = connect(
  null,
  dispatch => ({
    changeVisibility: value => dispatch(changeVisibility(value))
  })
)(Visibility);
```

We have two buttons `Visible` and `Hidden`. They both fire `CHANGE_VISIBILITY` action but the first one passes `true` as a value while the second one `false`. The `VisibilityConnected` component class gets created as a result of the wiring done via Redux's `connect`. Notice that we pass `null` as `mapStateToProps` because we don't need any of the data in the store. We just need to `dispatch` an action.

The second component is slightly more complicated. It is named `Counter` and renders two buttons and the counter value.

```js
function Counter({ value, add, subtract }) {
  return (
    <div>
      <p>Value: { value }</p>
      <button onClick={ add }>Add</button>
      <button onClick={ subtract }>Subtract</button>
    </div>
  );
}

const CounterConnected = connect(
  state => ({
    value: getCounterValue(state)
  }),
  dispatch => ({
    add: () => dispatch(add()),
    subtract: () => dispatch(subtract())
  })
)(Counter);
```

We now need both `mapStateToProps` and `mapDispatchToProps` because we want to read data from the store and dispatch actions. Our component receives three props - `value`, `add` and `subtract`. 

The very last bit is an `App` component where we compose the application.

```js
  return (
    <div>
      <VisibilityConnected />
      { visible && <CounterConnected /> }
    </div>
  );
}
const AppConnected = connect(
  state => ({
    visible: getVisibility(state)
  })
)(App);
```

We again need to `connect` our component because we want to control the visibility of the counter. The `getVisibility` selector returns a boolean that indicates whether `CounterConnected` will be rendered or not.

## Final thoughts

Redux is a wonderful pattern. Over the years the JavaScript community developed the idea and enhanced it with couple of new terms. I think a typical redux application looks more like this:

![Redux architecture](redux-reallife.jpg)

*By the way we didn't mention the side effects management. It is a whole new story with its own ideas and solutions.*

We can conclude that Redux itself is a pretty simple pattern. It teaches very useful techniques but unfortunately it is very often not enough. Sooner or later we have to introduce more concepts/patterns. Which of course is not that bad. We just have to plan for it.
