# Redux

[Redux](https://redux.js.org/) เป็น Library ที่ทำตัวเป็นตัวเก็บ state และช่วยจัดการ data ภายใน application เปิดตัวครั้งแรกในงาน ReactEurope conference ([วิดีโอ](https://www.youtube.com/watch?v=xsSnOQynTHs)) ในปี 2015 โดย [Dan Abramov](https://twitter.com/dan_abramov) มีลักษณะการทำงานและการออกแบบคล้าย [สถาปัตยกรรม Flux](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-8/README.md#flux-architecture-and-its-main-characteristics)

<span class="new-page"></span>

## สถาปัตยกรรม Redux และลักษณะสำคัญ

![Redux architecture](./redux-architecture.jpg)

เช่นเดียวกับ [Flux](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-8/README.md) Redux มี view components (React) ที่คอย dispatch action โดยที่ action เดียวกันสามารถถูก dispatch มาจากส่วนไหนของระบบก็ได้ ยกตัวอย่างเช่น การเรียก bootstrap เป็นต้น action ที่ถูก dispatch จะถูกส่งตรงไปยัง store ซึ่งมีแค่ตัวเดียวเท่านั้นใน Redux ซึ่งสิ่งที่ Redux ไม่เหมือนกับ Flux คือส่วนที่จะตัดสินใจว่า data ของเราจะเปลี่ยนไปอย่างไรนั้นขึ้นอยู่กับ reducers ที่เป็น pure functions เมื่อ store ได้รับ action reducers จะทำการรับ current state และ action ที่ถูกส่งเข้ามา เพื่อคำนวณและสร้าง state ถัดไป โดยอิงหลัก immutable store จะรับช่วงต่อและเปลี่ยนค่า state ภายใน store สุดท้าย React component ที่ดึง data มาจาก store ก็จะถูก re-render

Concept ของ Redux ค่อนข้างตรงไปตรงมาโดยยึดหลัก [one-direction data flow](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-7/README.md) เรามาเริ่มจาก แนะนำส่วนประกอบต่างๆ และตัวช่วยใน Redux pattern กันดีกว่า

### Actions

โดยทั่วไปแล้ว action ใน Redux เป็นเพียง object ที่ประกอบไปด้วย property ที่ชื่อว่า `type` ส่วน property อื่น ๆ ใน object คือ data ที่เกี่ยวข้องกับบริบทของ action นั้นๆ และไม่เกี่ยวข้องกับ pattern ของ Redux แต่อย่างใด ยกตัวอย่างเช่น

```js
const CHANGE_VISIBILITY = 'CHANGE_VISIBILITY';
const action = {
  type: CHANGE_VISIBILITY,
  visible: false
}
```
ถือว่าเป็นแบบอย่างที่ดีที่เราสร้าง constant อย่าง `CHANGE_VISIBILITY` เป็น action type ซึ่งยังมี tools หรือ libraries หลาย ๆ อย่างที่รองรับ Redux ที่มีวีธีเรียกใช้ที่สะดวกโดยการส่ง action type อย่างเดียว

ส่วนของ property `visible` เป็น metadata ที่เราได้กล่าวถึง ซึ่งไม่ได้ถูกใช้ใน Redux เป็นเพียงแค่ข้อมูลที่ใช้ใน application เท่านั้น

ทุก ๆ ครั้งที่เราต้องการ dispatch method เราต้องใช้ object ซึ่งมันเป็นการยุ่งยากถ้าจะต้องมาเขียนมันซ้ำแล้วซ้ำเล่า จึงเป็นที่มาของ *action creators* ที่เป็น function ที่ return ค่า object และอาจจะรับหรือไม่รับ argument ที่เกี่ยวข้องกับ action นั้นเพิ่มก็ได้ ยกตัวอย่างเช่น action creator ของ action ด้านบน จะมีหน้าตาตามด้านล่าง

```js
const changeVisibility = visible => ({
  type: CHANGE_VISIBILITY,
  visible
});

changeVisibility(false);
// { type: CHANGE_VISIBILITY, visible: false }
```

จะสังเกตได้ว่าเราทำการส่งค่าของ `visible` ผ่าน argument ทำให้เราไม่ต้องจดจำค่าจริงของ action type นั้น ซึ่งการใช้ตัวช่วยพวกนี้จะทำให้โค้ดของเราสั้นและง่ายต่อการอ่าน

### Store

Redux ได้เตรียมตัวช่วยอย่าง `createStore` ไว้สำหรับการสร้าง store โดย function มีลักษณะดังนี้

```js
import { createStore } from 'redux';

createStore([reducer], [initial state], [enhancer]);
```

เราได้กล่าวถึงไว้แล้วว่า reducer เป็น function ที่รับ current state และ action และ return ค่าเป็น state ใหม่ ต่อมา argument ที่สองคือ state เริ่มต้น ซึ่งมีประโยชน์ในการกำหนดค่า state เมื่อ application เริ่มทำงาน ซึ่ง feature นี้เป็นส่วนสำคัญของกระบวนการทำ server-side rendering หรือ persistent experience ส่วน argument ที่สามคือ enhancer ไว้ใช้สำหรับเชื่อมต่อกับ third party API หรือ function ที่ไม่ได้มีใน Redux ยกตัวอย่างเช่น function ที่ไว้จัดการ async processes

store ที่สร้างขึ้นมาประกอบไปด้วย 4 method คือ `getState`, `dispatch`, `subscribe` และ `replaceReducer` ซึ่งตัวที่สำคัญที่สุดคือ `dispatch`

```js
store.dispatch(changeVisibility(false));
```

ด้านบนเป็นวิธีที่เราเรียกใช้ action creators โดยเราจะส่งผลลัพธ์ที่ได้จาก action creators ซึ่งก็คือ action object ไปยัง `dispatch` method ซึ่งจะถูกกระจายต่อไปยัง reducer แต่ละตัวที่อยู่ใน application

โดยทั่วไปแล้วใน React application เราจะไม่ค่อยได้ใช้ `getState` และ `subscribe` ตรงๆ เพราะเรามีตัวช่วย (เราจะอธิบายในส่วนต่อไป) ในการที่จะเชื่อมต่อ component ของเราเข้ากับ store และ `subscribe` อย่างมีประสิทธิภาพเมื่อมีการเปลี่ยนแปลง ในส่วนของ subscription เรายังได้รับ current state ทำให้ไม่ต้องเรียก `getState` เองอีกด้วย ส่วน `replaceReducer` เป็น API ที่ค่อนข้างซับซ้อน ใช้สำหรับเปลี่ยน reducer ที่กำลังถูก store ใช้งานอยู่ โดยส่วนตัวผมแล้ว ผมยังไม่มีโอกาสได้ใช้เลย

### Reducer

reducer เป็น function ที่เรียกได้ว่า *สวยงามที่สุด* ภายใน Redux แม้ก่อนหน้านี้ ผมจะเริ่มสนใจในการเขียน pure fuction ที่มีคุณสมบัติ immutability อยู่แล้ว แต่ Redux บังคับให้ผมต้องเขียน reducer มีคุณสมบัติที่สำคัญอยู่ 2 ข้อด้วยกัน หากขาดหายไปแล้ว pattern นี้ก็จะไม่สมบูรณ์

(1) ต้องเป็น pure function เท่านั้น หมายความว่า function ควรจะ return ค่าเดียวกันทุกครั้งหากมี input ที่เหมือนกัน

(2) ควรจะไม่มี side effects กล่าวคือไม่ควรมีการแก้ไขค่า global variable, การเรียก async function หรือ การใช้งาน promise

นี่คือตัวอย่างง่าย ๆ ของ counter reducer

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

จะเห็นว่าไม่มี side effects และเรา return object ตัวใหม่ทุกครั้ง เราสร้าง value ขึ้นมาใหม่ โดยอิงจาก state ก่อนหน้าและ action type ที่ถูกส่งเข้ามา

### การเชื่อมต่อกับ React components

ถ้าเราพูดถึง Redux ที่ใช้ใน React แล้วมักจะหมายถึง [react-redux](https://github.com/reactjs/react-redux) ซึ่งมีสองอย่างที่ช่วยเชื่อมต่อ Redux กับ components ของเรา

(1) `<Provider>` component - เป็น component ที่รับ store เข้ามาเพื่อทำให้ children node ใน React tree สามารถ access store ได้โดยผ่าน React's context API ตัวอย่างเช่น

```js
<Provider store={ myStore }>
  <MyApp />
</Provider>
```

โดยปกติแล้วเราจะประกาศใช้แค่ที่เดียวใน app

(2) `connect` function - เป็น function ที่ใช้ subcribe เพื่ออัพเดท store และ re-render component โดย function จะสร้าง [higher-order component](https://github.com/krasimir/react-in-patterns/blob/master/book/chapter-4/README.md#higher-order-component) ออกมา โดย function มีลักษณะดังนี้

```
connect(
  [mapStateToProps],
  [mapDispatchToProps],
  [mergeProps],
  [options]
)
```

`mapStateToProps` เป็น function ที่รับ current state และ return เป็น set ของ key-value (object) ที่จะถูกส่งไปยัง React component ในรูปของ props ยกตัวอย่างเช่น

```js
const mapStateToProps = state => ({
  visible: state.visible
});
```

`mapDispatchToProps` ทำหน้าที่คล้ายกับ mapStateToProps แต่จะรับ function `dispatch` แทน `state` ซึ่งตรงนี้เป็นที่ ๆ เราจะประกาศ prop สำหรับ dispatch action

```js
const mapDispatchToProps = dispatch => ({
  changeVisibility: value => dispatch(changeVisibility(value))
});
```

`mergeProps` เป็นตัวที่รวม `mapStateToProps` และ `mapDispatchToProps` เข้าด้วยกัน เป็นจุดสุดท้ายที่เปลี่ยนแปลงค่า props ก่อนจะส่งไปยัง component จากตัวอย่างด้านบน ถ้าเราต้องการที่จะทำ action สองอย่าง เราสามารถรวมมันเข้าด้วยกันเป็น props เดียวแล้วส่งไปยัง React ได้ `options` รับ setting ไว้สำหรับควบคุมการทำงานของ connect function

<br />

## สร้าง counter app ง่าย ๆ ด้วย Redux

เรามาเริ่มสร้าง counter app แบบง่ายๆ ที่ใช้ APIs จากด้านบนกัน

![Redux counter app example](./redux-counter-app.png)

ปุ่ม "Add" และ "Subtract" จะเป็นตัวเปลี่ยนแปลงค่าที่อยู่ใน store ของเรา ส่วนปุ่ม "Visible" และ "Hidden" จะเป็นตัวควบคุมการแสดงค่า 

### การออกแบบ Actions

สำหรับผมแล้ว ทุก feature ใน Redux จะเริ่มต้นด้วยการออกแบบ action types และประกาศสิ่งที่จะเก็บใน state ในกรณีนี้เรามี 3 operation คือ adding, subtracting และ จัดการ visibility ดังนั้นเราจะมาเริ่มจาก: 

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
### Store และ Reducers

ยังมีบางอย่างที่เรายังไม่ได้พูดถึงตอนที่เราอธิบายเรื่อง store กับ reducer โดยปกติแล้วเราจะมี reducer มากกว่าหนึ่งตัว เพราะเราต้องการที่จะแยกจัดการหลาย ๆ อย่าง เรามี store อยู่ตัวเดียวอยู่แล้วและตามทฤษฏีแล้ว state ก็จะมีแค่ตัวเดียวเหมือนกัน โดยส่วนใหญ่ application ที่รันบน production จะมี state ที่ถูกแบ่งเป็นส่วน ๆ แสดงให้เห็นถึงแต่ละส่วนของระบบ ในตัวอย่างเรามีส่วนของ counting และ visibility ซึ่งเราสามารถสร้าง state ได้ตามนั้นเลย

```js
const initialState = {
  counter: {
    value: 0
  },
  visible: true
};
```

เราต้องสร้าง reducer แต่ละส่วนแยกกัน ซึ่งทำให้โค้ดของเรามีความยืดหยุ่นและอ่านง่ายมากขึ้น ลองคิดดูว่าถ้าเรามี app ที่มีขนาดใหญ่ที่มีการแบ่ง state มากกว่าสิบส่วน มันคงเป็นการยากที่เราจะต้องจัดการมันอยู่บน function เดียว

Redux มาพร้อมกับ function `combineReducers` ที่ทำให้เราสนใจเฉพาะ state ย่อยในแต่ละส่วน และระบุ reducer ให้ state นั้น

```js
import { createStore, combineReducers } from 'redux';

const rootReducer = combineReducers({
  counter: function A() { ... },
  visible: function B() { ... }
});
const store = createStore(rootReducer);
```

โดย function `A` จะรับเฉพาะส่วน `counter` จาก state และต้อง return ค่าเฉพาะส่วนนั้น เช่นเดียวกับ `B` ที่จะรับ boolean (ค่าของ `visible`) และต้อง return boolean เท่านั้น

สำหรับ reducer ในส่วนของ counter ควรจะทำงานเมื่อรับ action `ADD` และ `SUBTRACT` มาเพื่อคำนวณหาค่า `counter` state ใหม่

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

สำหรับ reducer ในส่วนของ visibility นั้นจะมีลักษณะคล้าย ๆ กัน เว้นแต่จะรับ action `CHANGE_VISIBILITY` แทน

```js
const visibilityReducer = function (state, action) {
  if (action.type === CHANGE_VISIBILITY) {
    return action.visible;
  }
  return true;
};
```

และสุดท้ายแล้วเราจะส่ง reducer ทั้งสองตัวไปยัง `combineReducer` เพื่อที่จะสร้างเป็น `rootReducer`

```js
const rootReducer = combineReducers({
  counter: counterReducer,
  visible: visibilityReducer
});
```

### Selectors

ก่อนจะเริ่มในส่วนถัดไป React components ที่เราได้กล่าวไว้ในส่วน concept ของ *selector* จาก section ที่แล้ว เรารู้แล้วว่า state ของเราถูกแบ่งออกเป็นหลาย ๆ ส่วน เราได้ให้ reducer จัดการเกี่ยวกับการ update data แต่เมื่อไหร่ที่มีการเรียก data เรายังคงมี state object ตัวเดียวอยู่ ซึ่งส่วนนี้ selector จะเป็นตัวช่วยจัดการ โดย selector จะเป็น function ที่รับ state object และเลือก return เฉพาะ data ที่เราต้องการ ยกตัวอย่างเช่น app เล็ก ๆ ของเราต้องการแค่ selector สองตัวนี้

```js
const getCounterValue = state => state.counter.value;
const getVisibility = state => state.visible;
```

counter app เล็กเกินกว่าที่จะเห็นประสิทธิภาพจริง ๆ ของการเขียนตัวช่วยพวกนี้ได้ แต่ใน project ใหญ่ ๆ จะต่างกันกันมาก ไม่ใช่แค่เพียงการที่เขียนโค้ดน้อยลงหรืออ่านง่าย เพราะ selector อาจมาพร้อมกับส่วนอื่น ๆ ที่อาจจะมี logic อยู่เนื่องจาก selector สามารถเข้าถึง state ได้ทั้งหมดทำให้สามารถใส่ business logic เพื่อตอบคำถามอย่างเช่น "user มีสิทธิ์ที่จะทำ X ในขณะที่อยู่หน้า Y ได้หรือไม่" ซึ่งสามารถจัดการได้ใน selector เดียว

### React components

เรามาเริ่มจัดการกับ UI และส่วนจัดการ visibility ของ counter กันก่อน

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

เราต้องการปุ่มสองปุ่มคือ `Visible` กับ `Hidden` ซึ่งทั้งสองปุ่มจะส่ง action `CHANGE_VISIBILITY` แต่ปุ่ม Visible จะส่งค่า `true` ส่วนปุ่ม Hidden จะส่งค่า `false` โดยที่ component class `VisibilityConnected` จะถูกสร้างมาจากการเชื่อมต่อ Redux ด้วย `connect` สังเกตว่าเราส่งค่า `null` เป็นแทน `mapStateToProps` เพราะว่าเราไม่ได้ต้องการ data อะไรจาก store เราแค่ต้องการ `dispatch` action เท่านั้น

component ที่สองจะซับซ้อนขั้นมากเล็กน้อย โดยที่มันมีชื่อว่า `Counter` และ render ปุ่มสองปุ่มและตัวแสดง counter

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

คราวนี้เราต้องส่งทั้ง `mapStateToProps` และ `mapDispatchToProps` เพราะว่าเราต้องมีการอ่าน data จาก store และ dispatch action โดย component ของเราจะรับค่า props สามตัวคือ `value`, `add` และ `subtract`

และสุดท้าย `App` component ที่ ๆ เราจะสร้าง application

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

เราต้องเรียกใช้ `connect` กับ component อีกครั้ง เพราะเราต้องการที่จะควบคุม visibility ของ counter โดยที่ `getVisibility` selector จะ return ค่า boolean ที่จะเป็นตัวกำหนด `CounterConnected` ว่าจะ render หรือไม่

## ข้อคิด

Redux เป็น pattern ที่ดี หลายปีแล้วที่ JavaScript community พัฒนาแนวคิดและเพิ่มประสิทธิภาพในหลาย ๆ ด้าน ผมคิดว่ารูปแบบ redux application จะมีหน้าตาใกล้เคียงภาพต่อไปนี้

![Redux architecture](redux-reallife.jpg)

*อย่างไรก็ตาม เราไม่ได้กล่าวถึงเรื่องการจัดการ side effects ซึ่งถือว่าเป็นเรื่องใหม่ที่มีแนวคิดและวิธีแก้ปัญหาของมันเอง*

เราสามารถสรุปได้ว่า Redux นั้นเป็น pattern ที่เรียบง่าย แถมยังสอนเทคนิคที่มีประโยชน์มาก แต่ในบางครั้งก็ยังไม่พอ ไม่เร็วก็ช้าเราจะมีแนวคิดหรือ pattern ใหม่ๆ ซึ่งนั่นไม่ใช้เรื่องแย่ เราแค่ต้องเตรียมพร้อมสำหรับมัน
