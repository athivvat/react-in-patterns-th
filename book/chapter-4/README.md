# Composition

หนึ่งในความสามารถที่มีประโยชน์มากของ React ก็คือ composability โดยส่วนตัวผมยังไม่รู้จัก framework ไหนที่มีวีธีในการสร้างและรวม component ได้อย่างที่ React ทำได้เลย ซึ่งในบทนี้เราก็จะมาลองดูเทคนิค composition บางตัวที่ได้รับการพิสูจน์ว่าใช้งานได้ดีกันนะครับ

One of the biggest benefits of React is composability. I personally don't know a framework that offers such an easy way to create and combine components. In this section we will explore few composition techniques which proved to work well.

เริ่มจากตัวอย่างง่ายๆกันเลย สมมุติว่าเรามี application ที่มีส่วนของ header อยู่ และเราต้องการที่จะวาง navigation ไว้ข้างใน ในกรณีนี้เรามี React component อยู่สามตัว ได้แก่ `App`, `Header`, และ `Navigation` โดยทั้งสามตัวต้องอยู่ในสภาพซ้อนทับกัน (nested) ซึ่งสามารถแสดง dependency ได้ดังนี้:

Let's get a simple example. Let's say that we have an application with a header and we want to place a navigation inside. We have three React components - `App`, `Header` and `Navigation`. They have to be nested into each other so we end up with the following dependencies:

```js
<App> -> <Header> -> <Navigation>
```

วิธีการง่ายๆในการรวม component เหล่านี้ก็คือการอ้างถึง component ในที่ต่างๆที่เราจะต้องใช้

The trivial approach for combining these components is to reference them in the places where we need them.

```js
// app.jsx
import Header from './Header.jsx';

export default function App() {
  return <Header />;
}

// Header.jsx
import Navigation from './Navigation.jsx';

export default function Header() {
  return <header><Navigation /></header>;
}

// Navigation.jsx
export default function Navigation() {
  return (<nav> ... </nav>);
}
```

อย่างไรก็ตาม หากเราทำตามวิธีการข้างต้น เราจะเจอกับปัญหาสองสามอย่าง:

However, by following this approach we introduced couple of problems:

* เราสามารถมองว่า `App` เป็นที่ที่เราจะทำ composition สำหรับ component หลักๆของเรา แต่ทั้งนี้ `Header` อาจจะมี element อื่นๆอย่างโลโก้ ช่องค้นหา หรือสโลแกนอยู่ด้วย คงจะดีมากหาก element เหล่านั้นถูกส่งผ่านมาจาก component `App` เพื่อที่เราจะไม่ต้องสร้าง hard-coded dependency ขึ้น แล้วในกรณีที่เราต้องการ component `Header` ตัวเดิมแต่ไม่ต้องการ `Navigation` ล่ะ? เราไม่สามารถแก้ปัญหานี้ได้ง่ายๆเนื่องจาก component ทั้งสองนั้นผูกกันอยู่
* การทดสอบจะเป็นเรื่องยากมาก โดยเราอาจจะมี business logic บางส่วนอยู่ใน `Header` และเพื่อที่จะทดสอบ logic เหล่านั้นเราจำเป็นต้องสร้าง instance ของ component ขึ้นมา อย่างไรก็ตาม เนื่องจาก `Header` ทำการ import component อื่นๆเข้ามาด้วย เราจึงอาจจะจำเป็นต้องสร้าง instance ของ component พวกนั้นด้วย ซึงจะกลายเป็นว่าการทดสอบเป็นงานที่หนักมากไปเลย นอกจากนี้การทดสอบ `Header` อาจจะล้มเหลวเพราะความผิดพลาดภายใน `Navigation` ก็ได้ ซึ่งจะทำให้เกิดความเข้าใจผิดขึ้นด้วย *(หมายเหตุ: ในบางกรณี [shallow rendering](https://facebook.github.io/react/docs/test-utils.html#shallow-rendering) สามารถแก้ปัญหานี้ได้โดยการ render เฉพาะ `Header` โดยที่ไม่สนใจ component ลูกที่ซ้อนอยู่ข้างใน)*

* We may consider the `App` as a place where we do our main composition. The `Header` though may have other elements like a logo, search field or a slogan. It will be nice if they are passed somehow from the `App` component so we don't create a hard-coded dependencies. What if we need the same `Header` component but without the `Navigation`. We can't easily achieve that because we have the two bound tightly together.
* It's difficult to test. We may have some business logic in the `Header` and in order to test it we have to create an instance of the component. However, because it imports other components we will probably create instances of those components too and it becomes heavy to test. We may break our `Header` test by doing something wrong in the `Navigation` component which is totally misleading. *(Note: to some extent [shallow rendering](https://facebook.github.io/react/docs/test-utils.html#shallow-rendering) solves this problem by rendering only the `Header` without its nested children.)*

## Using React's children API
## การใช้ React's children API

ใน React นั้นมี prop ชื่อว่า [`children`](https://facebook.github.io/react/docs/multiple-components.html#children) ซึ่ง parent ใช้ในการอ่านหรือเข้าถึงลูกๆ โดย API นี้จะทำให้ Header ของเรานั้น dependency-free

In React we have the handy [`children`](https://facebook.github.io/react/docs/multiple-components.html#children) prop. That's how the parent reads/accesses its children. This API will make our Header agnostic and dependency-free:

```js
export default function App() {
  return (
    <Header>
      <Navigation />
    </Header>
  );
}
export default function Header({ children }) {
  return <header>{ children }</header>;
};
```

สังเกตุว่า ถ้าเราไม่ใช้ `{ children }` ใน `Header` จะทำให้ `Navigation` ไม่ถูก render

Notice also that if we don't use `{ children }` in `Header`, the `Navigation` component will never be rendered.

ตอนนี้การทดสอบก็ง่ายขึ้นมาแล้ว เพราะเราสามารถ render `Header` ด้วย `<div>` เปล่าๆได้ ซึ่งนี่จะทำให้ component อยู่แยกกันและทำให้เราสามารถโฟกัสไปแต่ละส่วนของ application ได้

It now becomes easier to test because we may render the `Header` with an empty `<div>`. This will isolate the component and will let us focus on one piece of our application.

## Passing a child as a prop
## การส่งผ่าน child ในรูปของ prop

ทุกๆ component ใน React จะสามารถรับ prop ได้ อย่างที่เราได้กล่าวไปแล้วว่าไม่มีกฏไหนบอกว่า prop เหล่านี้จะต้องเป็นอะไร ดังนั้นเราจึงสามารถส่ง component อื่นๆเข้าไปเป็น prop ก็ได้

Every React component receives props. As we mentioned already there is no any strict rule about what these props are. We may even pass other components.

```js
const Title = function () {
  return <h1>Hello there!</h1>;
}
const Header = function ({ title, children }) {
  return (
    <header>
      { title }
      { children }
    </header>
  );
}
function App() {
  return (
    <Header title={ <Title /> }>
      <Navigation />
    </Header>
  );
};
```

เทคนิคนี้มีประโยชน์เมื่อ component อย่าง `Header` จำเป็นต้องตัดสินใจเกี่ยวกับ children แต่ไม่จำเป็นต้องรู้ว่าจริงๆแล้ว children คืออะไร ตัวอย่างง่ายๆก็เช่น visibility component ที่ซ่อน children ไว้ตามเงื่อนไขบางอย่าง

This technique is useful when a component like `Header` needs to take decisions about its children but don't bother about what they actually are. A simple example is a visibility component that hides its children based on a specific condition.

## Higher-order component

เป็นเวลานานมากแล้วที่ higher-order component เป็นวิธีที่นิยมใช้ในการ enhance element ใน React ซึ่ง component ชนิดนี้มีความคล้ายคลึงอย่างมากกับ [decorator design pattern](http://robdodson.me/javascript-design-patterns-decorator/) เพราะ component เหล่านี้ wrap และ enhance เช่นกัน

For a long period of time higher-order components were the most popular way to enhance and compose React elements. They look really similar to the [decorator design pattern](http://robdodson.me/javascript-design-patterns-decorator/) because we have component wrapping and enhancing.

ในทางเทคนิค higher-order component ปกติจะเป็น function ที่รับ original component และ return ตัวมันในรูปแบบที่ถูก enhance หรือ populate ตัวอย่างเล็กๆก็เช่น:

On the technical side the higher-order component is usually a function that accepts our original component and returns an enhanced/populated version of it. The most trivial example is as follows:

```js
var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    render() {
      return (
        <Component {...this.props} />
      )
    }
  };

var OriginalTitle = () => <h1>Hello world</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);

class App extends React.Component {
  render() {
    return <EnhancedTitle />;
  }
};
```

สิ่งแรกที่ higher-order component ทำก็คือ render original component โดย good practice คือการทำ proxy pass `props` เข้าไป วิธีนี้จะทำให้เรายังสามารถเก็บ input ของ original component ไว้ได้ และเนื่องจากเราสามารถควบคุม input ของ component อยู่ เราจึงอาจจะส่งอะไรเข้าไปก็ได้ อะไรที่ปกติแล้ว component ไม่สามารถเข้าถึงได้ ตรงส่วนนี้เองที่ถือเป็นประโยชน์ใหญ่ๆข้อแรก ลองสมมุติว่าเรามี configuration setting ที่ `OriginalTitle` ต้องใช้:

The very first thing that the higher-order component does is to render the original component. It's a good practice to proxy pass the `props` to it. This way we will keep the input of our original component. And here it comes the first big benefit of this pattern - because we control the input of the component we may send something that the component usually has no access to. Let's say that we have a configuration setting that `OriginalTitle` needs:

```js
var config = require('path/to/configuration');

var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    render() {
      return (
        <Component
          {...this.props}
          title={ config.appTitle }
        />
      )
    }
  };

var OriginalTitle  = ({ title }) => <h1>{ title }</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);
```

Knowledge ของ `appTitle` นั้นถูกซ่อนอยู่ใน higher-order component และ `OriginalTitle` รู้แค่เพียงว่ามันรับ `prop` ที่เรียกว่า `title` เข้ามา โดย `OriginalTitle` ไม่รู้เลยว่า prop ดังกล่าวมาจากไฟล์ configuration ซึ่งนั่นก็คือข้อได้เปรียบอย่างใหญ่หลวงเพราะตอนนี้เราสามารถแยก block การทำงานออกจากกันได้ นอกจากนี้ยังช่วยเรื่องการทดสอบ component ด้วย เพราะเราสามารถสร้าง mock ได้อย่างง่ายดาย

The knowledge for the `appTitle` is hidden into the higher-order component. `OriginalTitle` knows only that it receives a `prop` called `title`. It has no idea that this is coming from a configuration file. That's a huge advantage because it allows us to isolate blocks. It also helps with the testing of the component because we can create mocks easily.

อีกหนึ่งคุณลักษณะของ pattern นี้ก็คือ เรามี buffer สำหรับเพิ่ม logic เข้าไปได้อีกด้วย ตัวอย่างเช่น ถ้า `OriginalTitle` ต้องใช้ข้อมูลซึ่งมาจาก remote server เราอาจจะทำการ query ข้อมูลนี้ใน higher-order component และส่งมันกลับไปในรูปของ prop ก็ได้

Another characteristic of this pattern is that we have a nice buffer for additional logic. For example, if our `OriginalTitle` needs data also from a remote server. We may query this data in the higher-order component and again send it as a prop.

<span class="new-page"></span>

```js
var enhanceComponent = (Component) =>
  class Enhance extends React.Component {
    constructor(props) {
      super(props);

      this.state = { remoteTitle: null };
    }
    componentDidMount() {
      fetchRemoteData('path/to/endpoint').then(data => {
        this.setState({ remoteTitle: data.title });
      });
    }
    render() {
      return (
        <Component
          {...this.props}
          title={ config.appTitle }
          remoteTitle={ this.state.remoteTitle }
        />
      )
    }
  };

var OriginalTitle  = ({ title, remoteTitle }) =>
  <h1>{ title }{ remoteTitle }</h1>;
var EnhancedTitle = enhanceComponent(OriginalTitle);
```

เช่นเคย `OriginalTitle` รู้แค่ว่ามันรับ prop เข้ามาสองตัวและต้อง render ต่อกัน สิ่งเดียวที่ `OriginalTitle` ต้องกังวลคือข้อมูลนั้นมีหน้าตาอย่าไร ไม่ใช่ว่ามาจากไหนและมาได้ยังไง

Again, the `OriginalTitle` knows that it receives two props and has to render them next to each other. Its only concern is how the data looks like not where it comes from and how.

*[Dan Abramov](https://github.com/gaearon) ได้พูดไว้[อย่างน่าสนใจ](https://github.com/krasimir/react-in-patterns/issues/12)ว่า การขั้นตอนการสร้าง higher-order component (ซึ่งก็คือการเรียก function อย่าง `enhanceComponent`) นั้นควรจะเกิดขึ้นที่ระดับ component definition หรือพูดอีกอย่างก็คือ เป็น bad practice หากจะสร้าง higher-order component ข้างใน component ของ React อีกตัว เพราะจะทำให้ช้าและอาจเกิด performance issue ได้

*[Dan Abramov](https://github.com/gaearon) made a really [good point](https://github.com/krasimir/react-in-patterns/issues/12) that the actual creation of the higher-order component (i.e. calling a function like `enhanceComponent`) should happen at a component definition level. Or in other words, it's a bad practice to do it inside another React component because it may be slow and lead to performance issues.*

<br /><br />

## Function as a children, render prop
## การใช้ function เป็น children และ render prop

ไม่กี่เดือนก่อน React community เริ่มที่จะเปลี่ยนทิศทางความสนใจแล้ว จนถึงตอนนี้ ในตัวอย่างของเรา `children` prop คือ React component ตัวหนึ่ง อย่างไรก็ตามยังมีอีก pattern หนึ่งที่ตอนนี้กำลังได้รับความนิยม นั่นก็คือ `children` prop ในรูปแบบของ JSX expression เรามาเริ่มด้วยกันส่งผ่าน object ธรรมดาๆดูก่อน

Last couple of months the React community started shifting in an interesting direction. So far in our examples the `children` prop was a React component. There is however a new pattern gaining popularity in which the same `children` prop is a JSX expression. Let's start by passing a simple object.

```js
function UserName({ children }) {
  return (
    <div>
      <b>{ children.lastName }</b>,
      { children.firstName }
    </div>
  );
}

function App() {
  const user = {
    firstName: 'Krasimir',
    lastName: 'Tsonev'
  };
  return (
    <UserName>{ user }</UserName>
  );
}
```

นี่อาจจะดูแปลก แต่ในความเป็นจริงนั้นเป็นวิธีที่ทรงพลังมาก ตัวอย่างเช่น เมื่อเรามี knowledge บางอย่างใน component แม่ และไม่จำเป็นต้องส่ง knowledge นั้นให้ลูก ตัวอย่างข้างล่างนี้พิมพ์ลิสต์ของ TODO ออกมา โดย `App` component เก็บข้อมูลทั้งหมดไว้และรู้วิธีดูว่า TODO นี้เสร็จหรือยัง ซึ่ง `TodoList` component นั้นแค่หุ้ม HTML markup ไว้

This may look weird but in fact is really powerful. Like for example when we have some knowledge in the parent component and don't necessary want to send it down to children. The example below prints a list of TODOs. The `App` component has all the data and knows how to determine whether a TODO is completed or not. The `TodoList` component simply encapsulate the needed HTML markup.

<br /><br /><br /><br />

```js
function TodoList({ todos, children }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ children(todo) }</li>
        ))
      }</ul>
    </section>
  );
}

function App() {
  const todos = [
    { label: 'Write tests', status: 'done' },
    { label: 'Sent report', status: 'progress' },
    { label: 'Answer emails', status: 'done' }
  ];
  const isCompleted = todo => todo.status === 'done';
  return (
    <TodoList todos={ todos }>
      {
        todo => isCompleted(todo) ?
          <b>{ todo.label }</b> :
          todo.label
      }
    </TodoList>
  );
}
```

สังเกตุวิธีที่ `App` component ไม่ได้เปิดเผย structure ของข้อมูลเลย และ `TodoList` ก็ไม่รู้ว่ามี `lebel` หรือ `status` property อยู่ด้วย

Notice how the `App` component doesn't expose the structure of the data. `TodoList` has no idea that there is `label` or `status` properties.

ซึ่ง pattern *render prop* นั้นก็เป็นเหมือนกัน ยกเว้นแต่ว่าเราใช้ prop ไม่ใช่ `children` ในการ render todo

The so called *render prop* pattern is almost the same except that we use a prop and not `children` for rendering the todo.

<br /><br /><br />

```js
function TodoList({ todos, render }) {
  return (
    <section className='main-section'>
      <ul className='todo-list'>{
        todos.map((todo, i) => (
          <li key={ i }>{ render(todo) }</li>
        ))
      }</ul>
    </section>
  );
}

return (
  <TodoList
    todos={ todos }
    render={
      todo => isCompleted(todo) ?
        <b>{ todo.label }</b> : todo.label
    } />
);
```

โดย pattern *function as children* และ *render prop* เมื่อไม่นานมานี้ได้กลายมาเป็นหนึ่งใน pattern ที่ผมชอบมาก ทั้งสองมอบ flexibility และช่วยในกรณีที่เราต้องการ reuse code ซึ่งทั้งสองยังเป็นวิธีการที่ทรงพลังในการ abstract code ส่วนที่สำคัญมากๆด้วย

These two patterns, *function as children* and *render prop* are probably one of my favorite ones recently. They provide flexibility and help when we want to reuse code. They are also a powerful way to abstract imperative code.

```js
class DataProvider extends React.Component {
  constructor(props) {
    super(props);

    this.state = { data: null };
    setTimeout(() => this.setState({ data: 'Hey there!' }), 5000);
  }
  render() {
    if (this.state.data === null) return null;
    return (
      <section>{ this.props.render(this.state.data) }</section>
    );
  }
}
```

`DataProvider` ไม่ reder อะไรเลยเมื่อครั้งแรกที่ถูก mount แต่ห้าวินาทีหลังจากที่เราอัพเดท state ของ component เราได้ทำการ render `<section>` ตามด้วยสิ่งที่ prop `render` return กลับมา ลองนึกภาพว่า component ตัวเดียวกันนี้ fetch ข้อมูลมาจาก remote server และเราต้องการจะแสดงผลก็ต่อเมื่อข้อมูลดังกล่าวมาถึงแล้วเท่านั้น

`DataProvider` renders nothing when first gets mounted. Five seconds later we update the state of the component and we render a `<section>` followed by what is `render` prop returning. Imagine that this same component fetches data from a remote server and we want to display it only when it is available.

```js
<DataProvider render={ data => <p>The data is here!</p> } />
```

ซึ่งเราได้ระบุว่าเราต้องการอะไร แต่ไม่ได้บอกว่าจะได้มาอย่างไร วิธีการดังกล่าวถูกซ่อนอยู่ภายใน `DataProvider` โดยในช่วงหลังๆ เราใช้ pattern นี้ในการทำงานที่เราต้องจำกัด UI บางส่วนไว้ที่ ผู้ใช้บางกลุ่มที่มี permission `read:products` แล้วเราจึงใช้ *render prop* pattern

We do say what we want to happen but not how. That is hidden inside the `DataProvider`. These days we used this pattern at work where we had to restrict some UI to certain users having `read:products` permissions. And we used the *render prop* pattern.

```js
<Authorize
  permissionsInclude={[ 'read:products' ]}
  render={ () => <ProductsList /> } />
```

ผลลัพธ์ที่ออกมานั้นดูสวยและยังอธิบายตัวเองไปในตัวได้อีกด้วย โดย `Authorize` ถูกส่งไปที่ identity provider และตรวจสอบว่า permission ของ user ปัจจุบันคืออะไร ถ้า user ได้รับอนุญาติให้อ่าน products ได้ เราก็จะ render `ProductList`

Pretty nice and self-explanatory in a declarative fashion. `Authorize` goes to our identity provider and checks what are the permissions of the current user. If he/she is allowed to read our products we render the `ProductList`.

## Final thoughts
## ทิ้งท้าย

เคยสงสัยมั้ยว่าทำไม HTML ถึงยังคงมีอยู่ ณ จุดนี้ HTML ถูกสร้างในยุกแรกของ Internet และเรายังใช้คงใช้กันอยู่จนถึงทุกวันนี้ นั่นเป็นเพราะ HTML มี composability สูงยังไงล่ะ React และ JSX เองก็ดูคล้ายกับ HTML ที่ได้รับ steroid เข้าไปในรูปแบบที่ว่าทั้งสองมี composibility ที่สูงเช่นกัน เพราะฉะนั้นจงทำให้แน่ใจว่าคุณได้เรียนรู้เทคนิค composition อย่างลึกซึ้ง เพราะนั่นคือหนึ่งในความสามรถที่มีประโยชน์ที่สุดของ React

Did you wonder why HTML is still here. It was created in the dawn of the internet and we still use it. That is because is highly composable. React and its JSX looks like HTML on steroids and as such it comes with the same capabilities. So, make sure that you master the composition because that is one of the biggest benefits of React.
