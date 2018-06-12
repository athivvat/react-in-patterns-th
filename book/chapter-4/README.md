# Composition

หนึ่งในความสามารถที่มีประโยชน์มากของ React ก็คือ composability โดยส่วนตัวผมยังไม่รู้จัก framework ไหนที่มีวีธีในการสร้างและรวม component ได้อย่างที่ React ทำได้เลย ซึ่งในบทนี้เราก็จะมาลองดูเทคนิค composition บางตัวที่ได้รับการพิสูจน์ว่าใช้งานได้ดีกันนะครับ

เริ่มจากตัวอย่างง่าย ๆ กันเลย สมมติว่าเรามี application ที่มีส่วนของ header อยู่ และเราต้องการที่จะวาง navigation ไว้ข้างใน ในกรณีนี้เรามี React component อยู่สามตัว ได้แก่ `App`, `Header`, และ `Navigation` โดยทั้งสามตัวต้องอยู่ในสภาพซ้อนทับกัน (nested) ซึ่งสามารถแสดง dependency ได้ดังนี้:

```js
<App> -> <Header> -> <Navigation>
```

วิธีการง่าย ๆ ในการรวม component เหล่านี้ก็คือการอ้างถึง component ในที่ต่าง ๆ ที่เราต้องการจะให้แต่ละ component ไปอยู่

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

* เราสามารถมองว่า `App` เป็นที่ที่เราจะทำ composition หลักได้ แต่ทั้งนี้ `Header` อาจจะมี element อื่น ๆ อย่างโลโก้ ช่องค้นหา หรือสโลแกนอยู่ด้วย ซึ่งคงดีมากหากด้วยวิธีการบางอย่าง element เหล่านี้สามารถถูกส่งมาจาก `App` component ได้ เพื่อที่เราจะได้ไม่ต้อง hard code ลงไป แล้วในกรณีที่เราต้องการ `Header` component ตัวเดิมแต่ไม่ต้องการ `Navigation` ล่ะ? จะเห็นว่าเราไม่สามารถแก้ปัญหานี้ได้ง่าย ๆ เนื่องจาก component ทั้งสองนั้นผูกกันอยู่
* การทดสอบจะกลายเป็นเรื่องยาก โดยเราอาจจะมี business logic บางส่วนอยู่ใน `Header` และเพื่อที่จะทดสอบ logic เหล่านั้นเราจำเป็นต้องสร้าง instance ของ component ขึ้นมา อย่างไรก็ตาม เนื่องจาก `Header` ทำการ import component อื่น ๆ เข้ามาด้วย เราจึงอาจจะจำเป็นต้องสร้าง instance ของ component พวกนั้นเช่นกัน ซึ่งจะทำให้การทดสอบกลายเป็นงานที่หนักมากไปเลย นอกจากนี้การทดสอบ `Header` อาจจะล้มเหลวเพราะความผิดพลาดภายใน `Navigation` ก็ได้ ซึ่งจะทำให้เกิดความเข้าใจผิดขึ้นด้วย *(หมายเหตุ: ในบางกรณี [shallow rendering](https://facebook.github.io/react/docs/test-utils.html#shallow-rendering) สามารถแก้ปัญหานี้ได้โดยการ render เฉพาะ `Header` และไม่สนใจลูก ๆ ที่ซ้อนอยู่ข้างใน)*

## การใช้ React's children API

ใน React นั้นมี prop ที่มีประโยชน์มากเรียกว่า [`children`](https://facebook.github.io/react/docs/multiple-components.html#children) ซึ่ง parent จะใช้ในการอ่านหรือเข้าถึงลูก ๆ โดย API นี้จะทำให้ Header ของเรานั้นกลายเป็น dependency-free ได้

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

สังเกตว่า ถ้าเราไม่ใช้ `{ children }` ใน `Header` จะทำให้ `Navigation` component ไม่ถูก render เลย

ซึ่งตอนนี้การทดสอบก็จะง่ายขึ้นมาแล้ว เพราะเราสามารถ render `Header` ด้วย `<div>` เปล่า ๆ ได้ นี่จะทำให้ component อยู่แยกกันและทำให้เราสามารถโฟกัสไปที่แต่ละส่วนของ application ได้

## การส่งลูกในรูปของ prop

ทุก ๆ component ใน React สามารถที่จะรับ prop ได้ และอย่างที่เราได้กล่าวไปแล้วว่าไม่มีกฏไหนบอกว่า prop เหล่านี้จะต้องเป็นอะไร เราอาจถึงขั้นส่ง component อื่น ๆ เข้าไปเป็น prop ก็ได้

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

เทคนิคนี้จะมีประโยชน์เมื่อ component อย่าง `Header` จำเป็นต้องทำการตัดสินใจเกี่ยวกับลูก ๆ แต่ไม่จำเป็นต้องรู้ว่าจริง ๆ แล้วลูกคืออะไร ตัวอย่างง่าย ๆ ก็เช่น visibility component ที่ซ่อนลูกไว้ตามเงื่อนไขบางอย่าง

## Higher-order component

เป็นเวลานานมากแล้วที่ higher-order component เป็นวิธีที่นิยมใช้ในการ enhance และประกอบ (compose) React element ซึ่ง component ชนิดนี้มีความคล้ายคลึงอย่างมากกับ [decorator design pattern](http://robdodson.me/javascript-design-patterns-decorator/) เพราะมีทั้ง wrap และ enhance เช่นกัน

ในทางเทคนิค higher-order component ปกติจะเป็น function ที่รับ component ดั้งเดิมและ return ตัวมันในรูปแบบที่ถูก enhance หรือ populate ตัวอย่างเล็ก ๆ ก็เช่น:

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

สิ่งแรกที่ higher-order component ทำก็คือ render component ดั้งเดิม โดย good practice คือการทำ proxy pass `props` เข้าไป วิธีนี้จะทำให้เรายังสามารถเก็บ input ของ component ดั้งเดิมไว้ได้ และเนื่องจากเราควบคุม input ของ component อยู่ เราจึงอาจจะส่งอะไรเข้าไปก็ได้ อะไรที่ปกติแล้ว component ไม่สามารถเข้าถึงได้ ตรงส่วนนี้เองที่ถือเป็นประโยชน์ใหญ่ ๆ ข้อแรก ลองสมมติว่าเรามีค่า config ที่ `OriginalTitle` ต้องใช้:

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

ค่าของ `appTitle` นั้นถูกซ่อนอยู่ใน higher-order component โดย `OriginalTitle` จะรู้เพียงแค่ว่ามันรับ `prop` ที่เรียกว่า `title` เข้ามาและไม่รู้เลยว่า prop ดังกล่าวมาจากไฟล์ configuration ซึ่งนั่นก็คือข้อได้เปรียบที่ใหญ่มากข้อหนึ่ง เพราะตอนนี้เราสามารถแยก block การทำงานออกจากกันได้และยังช่วยเรื่องการทำการทดสอบ component ด้วย เนื่องจากเราสามารถสร้าง mock ได้อย่างง่ายดายแล้ว

อีกหนึ่งคุณลักษณะของ pattern นี้ก็คือ เรามี buffer สำหรับใส่ logic เพิ่มเติมเข้าไปได้อีกด้วย ตัวอย่างเช่น ถ้า `OriginalTitle` ต้องใช้ข้อมูลซึ่งมาจาก remote server เราอาจจะทำการ query ข้อมูลนี้ใน higher-order component และส่งกลับไปในรูปของ prop ก็ได้

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

*[Dan Abramov](https://github.com/gaearon) ได้พูดไว้[อย่างน่าสนใจ](https://github.com/krasimir/react-in-patterns/issues/12)ว่า ขั้นตอนการสร้าง higher-order component (ซึ่งก็คือการเรียก function อย่าง `enhanceComponent`) นั้นควรจะเกิดขึ้นที่ระดับนิยามของ component (component definition level) หรือพูดอีกอย่างก็คือ เป็น bad practice หากจะสร้าง higher-order component ข้างใน React component อีกตัว เพราะจะทำให้ช้าและอาจนำไปสู่ performance issue ได้*

<br /><br />

## การใช้ function เป็น children และ render prop

ในช่วงสองสามเดือนหลังนี้ React community เริ่มที่จะเปลี่ยนทิศทางความสนใจบ้างแล้ว จนถึงตอนนี้ ในตัวอย่างของเรา `children` prop คือ React component ตัวหนึ่ง อย่างไรก็ตามยังมีอีก pattern หนึ่งที่กำลังได้รับความนิยมเพิ่มขึ้น นั่นก็คือ `children` prop ตัวเดิมที่กลายมาอยู่ในรูปของ JSX expression เรามาเริ่มด้วยการลองส่ง object ธรรมดา ๆ กันดูก่อนนะครับ

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

ตัวอย่างนี้อาจจะดูแปลกอยู่ซักหน่อย แต่จริง ๆ แล้วเป็นวิธีที่ใช้งานได้ดีมาก เช่นในกรณีที่เรามี knowledge บางอย่างใน component แม่ และไม่อยากที่จะต้องส่ง knowledge นั้นให้ลูก ตัวอย่างข้างล่างนี้พิมพ์ลิสต์ของ TODO ออกมา โดย `App` component จะเก็บข้อมูลทั้งหมดไว้และรู้วิธีตรวจสอบว่า TODO นี้เสร็จหรือยัง ซึ่ง `TodoList` component นั้นทำหน้าที่แค่ห่อหุ้ม HTML markup ที่จำเป็นไว้เท่านั้น

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

สังเกตวิธีที่ `App` component ไม่ได้เปิดเผยโครงสร้างของข้อมูลเลย และ `TodoList` ก็ไม่รู้ว่ามี `label` หรือ `status` property อยู่ด้วย

โดย *render prop* pattern นั้นก็เป็นเช่นเดียวกัน ยกเว้นแต่ว่าเราใช้ prop ไม่ใช่ `children` ในการ render todo

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

ซึ่ง pattern ทั้งสองตัว ได้แก่ *function as children* และ *render prop* นั้นเมื่อไม่นานมานี้ได้กลายมาเป็นหนึ่งใน pattern โปรดของผมแล้ว โดยทั้งสองมอบ flexibility และช่วยในกรณีที่เราต้องการ reuse code และทั้งสองยังเป็นวิธีการที่ใช้ได้ดีมากในการ abstract code ส่วนที่สำคัญ ๆ อีกด้วย

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

`DataProvider` ไม่ได้ reder อะไรเลยเมื่อครั้งแรกที่ถูก mount แต่ห้าวินาทีหลังจากที่เราอัพเดท state ของ component เราได้ทำการ render `<section>` ตามด้วยสิ่งที่ prop `render` return กลับมา ลองนึกภาพว่า component ตัวเดียวกันนี้ดึงข้อมูลมาจาก remote server และเราต้องการจะแสดงผลก็ต่อเมื่อข้อมูลดังกล่าวมาถึงแล้วเท่านั้น

```js
<DataProvider render={ data => <p>The data is here!</p> } />
```

เราระบุว่าเราต้องการให้อะไรเกิดขึ้นแต่ไม่ใช่เกิดขึ้นได้อย่างไร วิธีการดังกล่าวถูกซ่อนอยู่ภายใน `DataProvider` โดยในช่วงหลัง ๆ เราใช้ pattern นี้ในงานที่เราต้องจำกัด UI บางส่วนไว้ที่ผู้ใช้บางกลุ่มที่มี permission `read:products` จากนั้นเราจึงใช้ *render prop* pattern

```js
<Authorize
  permissionsInclude={[ 'read:products' ]}
  render={ () => <ProductsList /> } />
```

ผลลัพธ์ที่ออกมานั้นดูสวยและยังอธิบายตัวเองไปในตัวได้อีกด้วย โดย `Authorize` ถูกส่งไปที่ identity provider และตรวจสอบว่า permission ของผู้ใช้ปัจจุบันคืออะไร ถ้าผู้ใช้ได้รับอนุญาติให้อ่าน products ได้ เราก็จะ render `ProductList`

## ข้อคิด

เคยสงสัยมั้ยว่าทำไม HTML ถึงยังคงมีอยู่จนถึงปัจจุบันนี้ทั้งที่ถูกสร้างในยุกแรกของ Internet แต่เราก็ยังคงใช้กันอยู่ นั่นเป็นเพราะ HTML มี composability ที่สูงมาก ซึ่ง React และ JSX เองก็ดูคล้ายกับ HTML ที่ได้รับ steroid เข้าไป ในรูปแบบที่ว่าทั้งสองมี composibility ที่สูงเช่นเดียวกัน ฉะนั้นจงทำให้แน่ใจว่าคุณได้เรียนรู้เทคนิค composition อย่างลึกซึ้งแล้ว เพราะนั่นคือหนึ่งในความสามรถที่มีประโยชน์ที่สุดของ React
