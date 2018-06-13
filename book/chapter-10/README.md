# การส่งต่อ dependency (Dependency injection)

Components หรือ modules ที่ถูกเขียนขึ้นมาส่วนใหญ่มักจะมี dependencies ติดมาด้วยเสมอ การที่เราสามารถจัดการ dependencies เหล่านั้น จึงเป็นส่วนสำคัญที่ทำให้โปรเจคของเราสำเร็จลุล่วงไปด้วยดี ปัจจุบัน มีเทคนิคชนิดหนึ่ง (หรือที่หลาย ๆ คนเรียกว่า *pattern*) ที่สามารถช่วยจัดการ dependencies ของเราได้ นั้นก็คือ [*dependency injection*](http://krasimirtsonev.com/blog/article/Dependency-injection-in-JavaScript)

ใน React เราสามารถมองได้ง่าย ๆ ว่าส่วนไหนที่ต้องการ dependency injector (หรือ ส่วนที่ต้องการใช้ dependency) ยกตัวอย่างเช่น application tree ด้านล่าง

```js
// Title.jsx
export default function Title(props) {
  return <h1>{ props.title }</h1>;
}

// Header.jsx
import Title from './Title.jsx';

export default function Header() {
  return (
    <header>
      <Title />
    </header>
  );
}

// App.jsx
import Header from './Header.jsx';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { title: 'React in patterns' };
  }
  render() {
    return <Header />;
  }
};
```

จากโค้ดตัวอย่าง จะเห็นได้ว่าค่าสตริง "React in patterns" จะต้องถูกส่งไปหา `Title` component โดยที่วิธีการตรง ๆ เลย คือการส่งค่าผ่าน props จาก `App` ไปยัง `Header` และจาก `Header` ไปยัง `Title` สำหรับ components สามตัวอาจจะไม่ใช้เรื่องแปลก แต่หาก components ที่เราต้องทำงานด้วยนั้น มี props ที่หลากหลาย และ มี component ที่ซ้อนกันหลาย ๆ ชั้น จะต้องมี components ระหว่างทางหลายตัวที่จะได้รับค่าไป เพียงเพื่อโยนไปให้ตัวลูกโดยที่ตัวเองไม่ได้ใช้

จากที่ผ่านมา เราได้เห็นแล้วว่า [higher-order component](https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components) นั้นสามารถใช้สำหรับการส่งค่าลงไปได้ เราจะมาลองใช้เทคนิคที่ว่ากับตัวแปร `title` ดู

```js
// inject.jsx
const title = 'React in patterns';

export default function inject(Component) {
  return class Injector extends React.Component {
    render() {
      return (
        <Component
          {...this.props}
          title={ title }
        />
      )
    }
  };
}

// -----------------------------------
// Header.jsx
import inject from './inject.jsx';
import Title from './Title.jsx';

var EnhancedTitle = inject(Title);
export default function Header() {
  return (
    <header>
      <EnhancedTitle />
    </header>
  );
}
```

ตอนนี้ตัวแปร `title` ได้ถูกซ่อนอยู่ในเลเยอร์ตรงกลาง (higher-order component) จากตรงนั้นมันจะถูกส่งไปหา `Title` component ผ่าน props โดยตรง ท่านี้แก้ปัญหาได้ครึ่งทาง เราไม่ต้องกังวลเรื่องการส่ง `title` ลงไปหลาย ๆ ชั้นอีกแล้ว แต่ยังมีปัญหาเรื่องที่ว่า เราจะทำยังไงให้ค่าวิ่งไปหา `inject.jsx`

## การใช้ React's context (เวอร์ชั่นก่อนหน้า 16.3)

*ในเวอร์ชั่น 16.3, ทีมผู้พัฒนา React ได้ เสนอ context API ตัวใหม่ และ สำหรับคนที่คิดว่าจะใช้ เวอร์ชั่น 16.3 หรือ มากกว่า สามารถข้ามส่วนนี้ไปได้เลย*

ในโลกของ React นั้น มีแนวคิดที่ชื่อว่า [*context*](https://facebook.github.io/react/docs/context.html) ซึ่ง context นั้นคือสิ่ง ๆ หนึ่งที่ React component ทุกตัวสามารถหยิบมาใช้ได้  แนวคิดของ context นั้นจะคล้าย ๆ กับ [event bus](https://github.com/krasimir/EventBus) สำหรับการส่งข้อมูล หรือ *store* อันหนึ่ง ที่สามารถเข้าถึงจากที่ไหนก็ได้

```js
// จุดที่เราทำการประกาศ context
var context = { title: 'React in patterns' };

class App extends React.Component {
  getChildContext() {
    return context;
  }
  ...
};
App.childContextTypes = {
  title: React.PropTypes.string
};

// จุดที่เราจะใช้ context
class Inject extends React.Component {
  render() {
    var title = this.context.title;
    ...
  }
}
Inject.contextTypes = {
  title: React.PropTypes.string
};
```

จะเห็นได้จากโค้ดด้านบน ว่าเราจะต้องประกาศ object context พร้อมตัวแปรที่เราจะใช้ ผ่าน `childContextTypes` และ `contextTypes` ถ้าเราไม่ประกาศ object `context` จะมาเป็น object เปล่า ๆ ซึ่งบางครั้งก็อาจจะทำให้รู้สึกหงุดหงิด ที่ต้องมานั่งใส่ตัวแปรหลาย ๆ ตัวลงไปในนั้น เพราะฉะนั้นวิธีการที่ดีคือ การเปลี่ยน `context` ให้มี interface ที่สามารถเก็บและส่งค่าได้ ดังตัวอย่างด้านล่าง

```js
// dependencies.js
export default {
  data: {},
  get(key) {
    return this.data[key];
  },
  register(key, value) {
    this.data[key] = value;
  }
}
```

ถ้านำกลับไปใช้กับตัวอย่างเดิม หน้าตาของ `App` component จะเป็นเหมือนด้านล่าง:

```js
import dependencies from './dependencies';

dependencies.register('title', 'React in patterns');

class App extends React.Component {
  getChildContext() {
    return dependencies;
  }
  render() {
    return <Header />;
  }
};
App.childContextTypes = {
  data: React.PropTypes.object,
  get: React.PropTypes.func,
  register: React.PropTypes.func
};
```

และ `Title` component ของเรา จะสามารถนำค่าจาก context ออกมาใช้ได้

```js
// Title.jsx
export default class Title extends React.Component {
  render() {
    return <h1>{ this.context.get('title') }</h1>
  }
}
Title.contextTypes = {
  data: React.PropTypes.object,
  get: React.PropTypes.func,
  register: React.PropTypes.func
};
```

ตามหลักการแล้ว เราไม่อยากที่จะนั่งประกาศ `contextTypes` ในทุก ๆ ครั้งที่เราอยากจะเข้าถึง context ในส่วนนี้เราสามารถนำ higher-order component มาครอบได้ และที่ดีกว่าคือเราสามารถเขียน utility function ที่มีความหมายชัดเจนมากกว่า และ ช่วยให้เราสามารถต่อ context ได้อย่างถูกต้อง ยกตัวอย่าง เช่น แทนที่เราจะเข้าถึง context ตรง ๆ ผ่าน `this.context.get('title')` เราสามารถขอ context ผ่าน higher-order component และให้ higher-order component ส่งค่ามาในรูปแบบของ props แทน ดังตัวอย่างด้านล่าง:


```js
// Title.jsx
import wire from './wire';

function Title(props) {
  return <h1>{ props.title }</h1>;
}

export default wire(Title, ['title'], function resolve(title) {
  return { title };
});
```

ฟังก์ชัน `wire` รับ React component, array ที่ประกอบด้วย dependencies (ที่`เชื่อมต่อ`เข้ากับ context แล้ว) ที่เราต้องการจะเรียกใช้ และ ฟังก์ชันที่ผู้เขียนชอบเรียกว่า `mapper` ซึ่งฟังก์ชันนี้จะรับค่ามาจาก context และ return ค่าในรูปแบบของ object โดยที่ object นั้นท้ายที่สุดแล้วจะถูกส่งให้ component ของเรา (`Title`) ในรูปแบบของ props ดังที่เห็นในตัวอย่างนี้ เรานำตัวแปร `title` ส่งเข้าไป
ในการเขียนแอปจริง ๆ ค่าที่ส่งเข้าไปอาจจะเป็น กลุ่มข้อมูลหลาย ๆ อัน, configuration หรือ อื่น ๆ

และนี้คือหน้าตาของฟังก์ชัน `wire`:

```js
export default function wire(Component, dependencies, mapper) {
  class Inject extends React.Component {
    render() {
      var resolved = dependencies.map(
        this.context.get.bind(this.context)
      );
      var props = mapper(...resolved);

      return React.createElement(Component, props);
    }
  }
  Inject.contextTypes = {
    data: React.PropTypes.object,
    get: React.PropTypes.func,
    register: React.PropTypes.func
  };
  return Inject;
};
```

`Inject` คือ higher-order component ที่สามารถเข้าถึง context และ นำค่าที่ประกาศไว้ใน array `dependencies` ออกมา ส่วนฟังก์ชัน `mapper` ทำหน้าที่รับ `context` เหล่านั้น และ ส่งเข้าไปหา component ของเราผ่าน props

## การใช้ React's context (เวอร์ชั่น 16.3 หรือ มากกว่า)

เป็นเวลาหลายปีที่ Facebook ไม่แนะนำให้ใช้ context API โดยให้เหตุผลไว้ใน official docs ว่า API นั้น ไม่เสถียร และ เสี่ยงต่อการเปลี่ยนแปลงในอนาคต และนั้นคือสิ่งที่เกิดขึ้นในปัจจุบัน ในเวอร์ชั่น 16.3 เราได้ API อันใหม่ ซึ่งผู้เขียนคิดว่า API นี้เป็นธรรมชาติมากขึ้น และ ใช้งานได้ง่ายกว่า

สมมุติว่าเราลองนำตัวอย่างเดิมมาใช้ ตัวอย่างที่เราต้องการส่งสตริงไปหา `<Title>` component

เราเริ่มโดยการสร้างไฟล์สำหรับการสร้าง context

```js
// context.js
import { createContext } from 'react';

const Context = createContext({});

export const Provider = Context.Provider;
export const Consumer = Context.Consumer;
```

ฟังก์ชัน `createContext` returns object ตัวหนึ่ง ที่มี properties ประกอบด้วย `.Provider` และ `.Consumer` โดยที่สองตัวนี้จริง ๆ แล้วคือ React class และสำหรับตัว Provider นั้น จะรับ context ผ่าน props ชื่อ `value` ในขณะที่ตัว consumer นั้น จะใช้สำหรับการเข้าถึงและอ่านค่า context ปกติแล้วสองตัวนี้จะอยู่คนละไฟล์ มันจึงเป็นความคิดที่ดี ที่จะสร้างที่ ๆ หนึ่งสำหรับการสร้างสองตัวนั้น

สมมุติว่า `App` component ของเรานั้นคือจุดสูงสุดของ application tree ข้างในนั้นเราจะทำการส่ง context เข้าไป

```js
import { Provider } from './context';

const context = { title: 'React In Patterns' };

class App extends React.Component {
  render() {
    return (
      <Provider value={ context }>
        <Header />
      </Provider>
    );
  }
};
```

ตอนนี้ components ที่โดยครอบและลูก ๆ ของมันได้ถูกแชร์ context อันเดียวกัน และ `<Title>` component คือตัวที่ต้องการสตริง `title` ตรงนี้จึงเป็นที่ ๆ เราจะนำ `<Consumer>` มาใช้

```js
import { Consumer } from './context';

function Title() {
  return (
    <Consumer>{
      ({ title }) => <h1>Title: { title }</h1>
    }</Consumer>
  );
}
```

*สังเกตุได้ว่า `Consumer` class ใช้ function as children (render prop) pattern สำหรับการส่ง context*

API อันใหม่นั้น ง่ายต่อการเข้าใจ ทั้งยังทำให้เราไม่ต้องใช้ boilerplate สำหรับตัว API นั้นค่อนข้างใหม่แต่ดูมีแนวโน้มที่ดี มันเปิดโอกาสให้เราเข้าถึงความเป็นไปได้ที่หลากหลายมากขึ้น

## การใช้ module system

ถ้าเราไม่ต้องการที่จะใช้ context ก็มีทางเลือกอื่นที่สามารถทำให้เราทำ injection ได้ โดยที่ทางเลือกนั้นอาจจะไม่เจาะจงไปที่ React แต่ก็ควรค่าแก่การกล่าวถึง หนึ่งในนั้นคือการใช้ module system

อย่างที่รู้ ๆ กันว่า ปกติแล้ว module system ใน Javascript นั้นมีกลไกการทำ caching โดยได้มีการโน้ตไว้ใน [Node's documentation](https://nodejs.org/api/modules.html#modules_caching):

> Modules นั้นจะถูก cached หลังจากที่มันถูกโหลดขึ้นมาครั้งแรก นั้นหมายความว่า ทุกครั้งที่เราเรียก required('foo') object อันเดิมจะถูกนำมาใช้เสมอถ้ามัน resolve ไปหาไฟล์อันเดิม

> การเรียกไปหา require('foo') หลาย ๆ ครั้ง จะไม่ทำให้โค้ดข้างใน foo module ถูกเรียกใหม่ซ้ำ ๆ นี้เป็นฟีเจอร์ที่สำคัญมากเพราะว่า "partially done" object (object ที่ยังรันไม่เสร็จ แต่ถูก require) จะถูก return ออกมาได้ และ ทำให้ transitive dependencies (dependency ตอนที่ modules require กันเอง) ถูกโหลดโดยไม่ทำให้เกิดลูป (cyclic dependency)

แล้วสิ่งเหล่านี้จะช่วยเราในการทำ injection อย่างไร ? มันช่วยเราได้เพราะ object ที่ถูก export ออกมานั้น จริง ๆ แล้วคือ [singleton](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#singletonpatternjavascript) และทุก module ที่ import ไฟล์นั้นเข้าไป ก็จะเข้าถึงอ็อบเจกต์ตัวเดียวกัน นั้นทำให้เราสามารถ ใส่ dependencies ของเราลงไป (`register`) และ นำออกมาจากไฟล์อื่น ๆ ได้ (`fetch`)

เราลองมาสร้างไฟล์ใหม่ชื่อ `di.jsx` ที่มีคอนเทนต์ตามด้านล่าง:

```js
var dependencies = {};

export function register(key, dependency) {
  dependencies[key] = dependency;
}

export function fetch(key) {
  if (dependencies[key]) return dependencies[key];
  throw new Error(`"${ key } is not registered as dependency.`);
}

export function wire(Component, deps, mapper) {
  return class Injector extends React.Component {
    constructor(props) {
      super(props);
      this._resolvedDependencies = mapper(...deps.map(fetch));
    }
    render() {
      return (
        <Component
          {...this.state}
          {...this.props}
          {...this._resolvedDependencies}
        />
      );
    }
  };
}
```

เราะจะเก็บ dependecies ไว้ในตัวแปร global ชื่อ `dependencies` (ตัวแปร global ในระดับ module ไม่ใช้ระดับแอปพลิเคชัน) หลังจากนั้นเราจะ export สองฟังก์ชันได้แก่ `register` และ `fetch` ที่จะทำหน้าที่เขียนและอ่านค่าต่าง ๆ โดยที่มันจะคล้าย ๆ กับการสร้าง setter และ getter ใน object ของ Javascript ต่อจากนั้นเราจะใช้ฟังก์ชัน `wire` ในการรับ React component และ return [higher-order component](https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components) ออกไป และใน constructor ของ component ที่อยู่ข้างในฟังก์ชัน wire เราจะทำการดึง dependencies ออกมา แล้วก็ส่งมันลงไปหา component ข้างใต้ที่กำลัง render ในรูปแบบของ props โดยที่เราจะทำตาม pattern เดิมที่เราอธิบายสิ่งที่เราต้องการ (`deps` argument) และดึง props ที่ต้องการออกมาผ่านฟังก์ชัน `mapper`

การที่เรามี `di.jsx` helper นั้นทำให้เราสามารถสร้าง dependencies ได้ที่จุดเริ่มต้นของแอปพลิเคชัน (`app.jsx`) และ ส่งมันลงไปในที่ ๆ เราต้องการได้ (`Title.jsx`)

<span class="new-page"></span>

```js
// app.jsx
import Header from './Header.jsx';
import { register } from './di.jsx';

register('my-awesome-title', 'React in patterns');

class App extends React.Component {
  render() {
    return <Header />;
  }
};

// -----------------------------------
// Header.jsx
import Title from './Title.jsx';

export default function Header() {
  return (
    <header>
      <Title />
    </header>
  );
}

// -----------------------------------
// Title.jsx
import { wire } from './di.jsx';

var Title = function(props) {
  return <h1>{ props.title }</h1>;
};

export default wire(
  Title,
  ['my-awesome-title'],
  title => ({ title })
);
```

*ถ้าเรามองที่ไฟล์ `Title.jsx` เราจะเห็นว่าตัว component และ ส่วนที่ทำการเชื่อมต่อนั้นสามารถอยู่คนละไฟล์ได้ ซึ่งท่านี้จะทำให้ตัว component และฟังก์ชัน mapper นั้นง่ายต่อการทำ unit test*

## ข้อคิด

Dependency injection นั้นเป็นปัญหาที่ยากโดยเฉพาะใน Javascript หลาย ๆ คนไม่คำนึงถึงว่าการทำ dependency management ที่เหมาะสมนั้น เป็นกระบวนการสำคัญในทุก development cycle และในส่วนของ JavaScript ecosystem นั้น มี tools ที่หลากหลายมานำเสนอให้เราอยู่เสมอ และ เรา developers ควรที่จะเลือกหยิบสิ่งที่ตอบโจทย์ต่อความต้องการของเรามากที่สุด
