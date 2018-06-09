# การส่งต่อ dependency (Dependency injection)

<!-- Big part of the modules/components that we write have dependencies. A proper management of these dependencies is critical for the success of the project. There is a technique (most people consider it a *pattern*) called [*dependency injection*](http://krasimirtsonev.com/blog/article/Dependency-injection-in-JavaScript) that helps solving the problem. -->

Components หรือ modules ที่ถูกเขียนขึ้นมาส่วนใหญ่มักจะมี dependencies ติดมาด้วยเสมอ การที่เราสามารถจัดการ dependencies เหล่านั้น จึงเป็นส่วนสำคัญที่ทำให้โปรเจคของเราสำเร็จลุล่วงไปด้วยดี ปัจจุบัน มีเทคนิคชนิดหนึ่ง (หรือที่หลายๆคนเรียกว่า *pattern*) ที่สามารถช่วยจัดการ dependencies ของเราได้ นั้นก็คือ [*dependency injection*](http://krasimirtsonev.com/blog/article/Dependency-injection-in-JavaScript)

<!-- In React the need of dependency injector is easily visible. Let's consider the following application tree: -->

ใน React เราสามารถมองได้ง่ายๆ ว่าส่วนไหนที่ต้องการ dependency injector (หรือ ส่วนที่ต้องการใช้ dependency) ยกตัวอย่างเช่น application tree ด้านล่าง

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

<!-- The string "React in patterns" should somehow reach the `Title` component. The direct way of doing this is to pass it from `App` to `Header` and then `Header` pass it down to `Title`. However, this may work for these three components but what happens if there are multiple properties and deeper nesting. Lots of components will act as proxy passing properties to their children. -->

จากโค้ดตัวอย่าง จะเห็นได้ว่าค่าสตริง "React in patterns" จะต้องถูกส่งไปหา `Title` component โดยที่วิธีการตรงๆเลย คือการส่งค่าผ่าน props จาก `App` ไปยัง `Header` และจาก `Header` ไปยัง `Title` สำหรับ components สามตัวอาจจะไม่ใช้เรื่องแปลก แต่หาก components ที่เราต้องทำงานด้วยนั้น มี props ที่หลากหลาย และ มี component ที่ซ้อนกันหลายๆชั้น จะต้องมี components ระหว่างทางหลายตัวที่จะได้รับค่าไป เพียงเพื่อโยนไปให้ตัวลูกโดยที่ตัวเองไม่ได้ใช้

<!-- We already saw how the [higher-order component](https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components) may be used to inject data. Let's use the same technique to inject the `title` variable: -->

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

<!-- The `title` is hidden in a middle layer (higher-order component) where we pass it as a prop to the original `Title` component. That's all nice but it solves only half of the problem. Now we don't have to pass the `title` down the tree but how this data reaches the `inject.jsx` helper. -->

ตอนนี้ตัวแปร `title` ได้ถูกซ่อนอยู่ในเลเยอร์ตรงกลาง (higher-order component) จากตรงนั้นมันจะถูกส่งไปหา `Title` component ผ่าน props โดยตรง ท่านี้แก้ปัญหาได้ครึ่งทาง เราไม่ต้องกังวลเรื่องการส่ง `title` ลงไปหลายๆชั้นอีกแล้ว แต่ยังมีปัญหาเรื่องที่ว่า เราจะทำยังไงให้ค่าวิ่งไปหา `inject.jsx`

<!-- ## Using React's context (prior v. 16.3) -->
## การใช้ React's context (ก่อนเวอร์ชั่น 16.3)

<!-- *In v16.3 React's team introduced a new version of the context API and if you are going to use that version or above you'd probably skip this section.* -->

*ในเวอร์ชั่น 16.3, ทีมผู้พัฒนา React ได้ เสนอ context API ตัวใหม่ และ สำหรับคนที่คิดว่าจะใช้ เวอร์ชั่น 16.3 หรือ มากกว่า สามารถข้ามส่วนนี้ไปได้เลย*

<!-- React has the concept of [*context*](https://facebook.github.io/react/docs/context.html). The *context* is something that every React component has access to. It's something like an [event bus](https://github.com/krasimir/EventBus) but for data. A single *store* which we access from everywhere. -->

ในโลกของ React นั้น มีแนวคิดที่ชื่อว่า [*context*](https://facebook.github.io/react/docs/context.html) ซึ่ง context นั้นคือสิ่งๆหนึ่งที่ React component ทุกตัวสามารถหยิบมาใช้ได้  แนวคิดของ context นั้นจะคล้ายๆกับ [event bus](https://github.com/krasimir/EventBus) สำหรับการส่งข้อมูล หรือ *store* อันหนึ่ง ที่สามารถเข้าถึงจากที่ไหนก็ได้

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

<!-- Notice that we have to specify the exact signature of the context object. With `childContextTypes` and `contextTypes`. If those are not specified then the `context` object will be empty. That may be a little bit frustrating because we may have lots of stuff to put there. That is why it is a good practice that our `context` is not just a plain object but it has an interface that allows us to store and retrieve data. For example: -->

จะเห็นได้จากโค้ดด้านบน ว่าเราจะต้องประกาศออบเจ็กต์ context พร้อมตัวแปรที่เราจะใช้ ผ่าน `childContextTypes` และ `contextTypes` ถ้าเราไม่ประกาศ ออบเจ็กต์ `context` จะมาเป็น ออบเจ็กต์เปล่าๆ ซึ่งบางครั้งก็อาจจะทำให้รู้สึกหงุดหงิด ที่ต้องมานั่งใส่ตัวแปรหลายๆตัวลงไปในนั้น เพราะฉะนั้นวิธีการที่ดีคือ การเปลี่ยน `context` ให้มี interface ที่สามารถเก็บและส่งค่าได้ ดังตัวอย่างด้านล่าง

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

<!-- Then, if we go back to our example, the `App` component may look like that: -->
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

<!-- And our `Title` component gets it's data through the context: -->
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

<!-- Ideally we don't want to specify the `contextTypes` every time when we need an access to the context. This detail may be wrapped again in a higher-order component. And even better, we may write an utility function that is more descriptive and helps us declare the exact wiring. I.e instead of accessing the context directly with `this.context.get('title')` we ask the higher-order component to get what we need and pass it as props to our component. For example: -->

ตามหลักการแล้ว เราไม่อยากที่จะนั่งประกาศ `contextTypes` ในทุกๆครั้งที่เราอยากจะเข้าถึง context ในส่วนนี้เราสามารถนำ higher-order component มาครอบได้ และที่ดีกว่าคือเราสามารถเขียน utility function ที่มีความหมายชัดเจนมากกว่า และ ช่วยให้เราสามารถต่อ context ได้อย่างถูกต้อง ยกตัวอย่าง เช่น แทนที่เราจะเข้าถึง context ตรงๆผ่าน `this.context.get('title')` เราสามารถขอ context ผ่าน higher-order component และให้ higher-order component ส่งค่ามาในรูปแบบของ props แทน ดังตัวอย่างด้านล่าง:


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

<!-- The `wire` function accepts a React component, then an array with all the needed dependencies (which are `register`ed already) and then a function which I like to call `mapper`. It receives what is stored in the context as a raw data and returns an object which is later used as props for our component (`Title`). In this example we just pass what we get - a `title` string variable. However, in a real app this could be a collection of data stores, configuration or something else.

Here is how the `wire` function looks like: -->

ฟังก์ชัน `wire` รับ React component, อาเรย์ที่ประกอบด้วย dependencies (ที่`เชื่อมต่อ`เข้ากับ context แล้ว) ที่เราต้องการจะเรียกใช้ และ ฟังก์ชันที่ผู้เขียนชอบเรียกว่า `mapper` ซึ่งฟังก์ชันนี้จะรับค่ามาจาก context และ return ค่าในรูปแบบของออบเจ็กต์ โดยที่ออบเจ็กต์นั้นท้ายที่สุดแล้วจะถูกส่งให้ component ของเรา (`Title`) ในรูปแบบของ props ดังที่เห็นในตัวอย่างนี้ เรานำตัวแปร `title` ส่งเข้าไป
ในการเขียนแอปจริงๆ ค่าที่ส่งเข้าไปอาจจะเป็น กลุ่มข้อมูลหลายๆอัน, configuration, หรือ อื่นๆ

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

<!-- `Inject` is a higher-order component that gets access to the context and retrieves all the items listed under `dependencies` array. The `mapper` is a function receiving the `context` data and transforms it to props for our component. -->

`Inject` คือ higher-order component ที่สามารถเข้าถึง context และ นำค่าที่ประกาศไว้ในอาเรย์ `dependencies` ออกมา ส่วนฟังก์ชัน `mapper` ทำหน้าที่รับ `context` เหล่านั้น และ ส่งเข้าไปหา component ของเราผ่าน props

<!-- ## Using React's context (v. 16.3 and above) -->
## การใช้ React's context (เวอร์ชั่น 16.3 หรือ มากกว่า)

<!-- For years the context API was not really recommended by Facebook. They mentioned in the official docs that the API is not stable and may change. And that is exactly what happened. In the version 16.3 we got a new one which I think is more natural and easy to work with. -->

เป็นเวลาหลายปีที่ Facebook ไม่แนะนำให้ใช้ context API โดยให้เหตุผลไว้ใน official docs ว่า API นั้น ไม่เสถียร และ เสี่ยงต่อการเปลี่ยนแปลงในอนาคต และนั้นคือสิ่งที่เกิดขึ้นในปัจจุบัน ในเวอร์ชั่น 16.3 เราได้ API อันใหม่ ซึ่งผู้เขียนคิดว่า API นี้เป็นธรรมชาติมากขึ้น และ ใช้งานได้ง่ายกว่า

<!-- Let's use the same example with the string that needs to reach a `<Title>` component. -->

สมมุติว่าเราลองนำตัวอย่างเดิมมาใช้ ตัวอย่างที่เราต้องการส่งสตริงไปหา `<Title>` component

<!-- We will start by defining a file that will contain our context initialization: -->

เราเริ่มโดยการสร้างไฟล์สำหรับการสร้าง context

```js
// context.js
import { createContext } from 'react';

const Context = createContext({});

export const Provider = Context.Provider;
export const Consumer = Context.Consumer;
```

<!-- `createContext` returns an object that has `.Provider` and `.Consumer` properties. Those are actually valid React classes. The `Provider` accepts our context in the form of a `value` prop. The consumer is used to access the context and basically read data from it. And because they usually live in different files it is a good idea to create a single place for their initialization. -->

ฟังก์ชัน `createContext` returns ออบเจ็กต์ตัวหนึ่ง ที่มี properties ประกอบด้วย `.Provider` และ `.Consumer` โดยที่สองตัวนี้จริงๆแล้วคือ React class และสำหรับตัว Provider นั้น จะรับ context ผ่าน props ชื่อ `value` ในขณะที่ตัว consumer นั้น จะใช้สำหรับการเข้าถึงและอ่านค่า context ปกติแล้วสองตัวนี้จะอยู่คนละไฟล์ มันจึงเป็นความคิดที่ดี ที่จะสร้างที่ๆหนึ่งสำหรับการสร้างสองตัวนั้น

<!-- Let's say that our `App` component is the root of our tree. At that place we have to pass the context. -->

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

<!-- The wrapped components and their children now share the same context. The `<Title>` component is the one that needs the `title` string so that is the place where we use the `<Consumer>`. -->

ตอนนี้ components ที่โดยครอบและลูกๆของมันได้ถูกแชร์ context อันเดียวกัน และ `<Title>` component คือตัวที่ต้องการสตริง `title` ตรงนี้จึงเป็นที่ๆเราจะนำ `<Consumer>` มาใช้

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

<!-- *Notice that the `Consumer` class uses the function as children (render prop) pattern to deliver the context.* -->

*สังเกตุได้ว่า `Consumer` class ใช้ function as children (render prop) pattern สำหรับการส่ง context*

<!-- The new API feels easier to understand and eliminates the boilerplate. It is still pretty new but looks promising. It opens a whole new range of possibilities. -->

API อันใหม่นั้น ง่ายต้องการเข้าใจ ทั้งยังทำให้เราไม่ต้องใช้ boilerplate สำหรับตัว API นั้นค่อนข้างใหม่แต่ดูมีแนวโน้มที่ดี มันเปิดโอกาศให้เราเข้าถึงความเป็นไปได้ที่หลากหลายมากขึ้น

<!-- ## Using the module system -->
## การใช้ module system

<!-- If we don't want to use the context there are a couple of other ways to achieve the injection. They are not exactly React specific but worth mentioning. One of them is using the module system. -->

ถ้าเราไม่ต้องการที่จะใช้ context ก็มีทางเลือกอื่นที่สามารถทำให้เราทำ injection ได้ โดยที่ทางเลือกนั้นอาจจะไม่เจาะจงไปที่ React แต่ก็ควรค่าแก่การกล่างถึง หนึ่งในนั้นคือการใช้ module system

<!-- As we know the typical module system in JavaScript has a caching mechanism. It's nicely noted in the [Node's documentation](https://nodejs.org/api/modules.html#modules_caching): -->

อย่างที่รู้ๆกันว่า ปกติแล้ว module system ใน Javascript นั้นมีกลไกการทำ caching โดยได้มีการโน้ตไว้ใน [Node's documentation](https://nodejs.org/api/modules.html#modules_caching):

<!-- > Modules are cached after the first time they are loaded. This means (among other things) that every call to require('foo') will get exactly the same object returned, if it would resolve to the same file. -->

> Modules นั้นจะถูก cached หลังจากที่มันถูกโหลดขึ้นมาครั้งแรก นั้นหมายความว่า ทุกครั้งที่เราเรียก required('foo') ออบเจ็ตก์อันเดิมจะถูกนำมาใช้เสมอถ้ามัน resolve ไปหาไฟล์อันเดิม

<!-- > Multiple calls to require('foo') may not cause the module code to be executed multiple times. This is an important feature. With it, "partially done" objects can be returned, thus allowing transitive dependencies to be loaded even when they would cause cycles. -->

> การเรียกไปหา require('foo') หลายๆครั้ง จะไม่ทำให้โค้ดข้างใน foo module ถูกเรียกใหม่ซ้ำๆ นี้เป็นฟีเจอร์ที่สำคัญมากเพราะว่า "partially done" ออบเจ็กต์ (ออบเจ็กต์ที่ยังรันไม่เสร็จ แต่ถูก require) จะถูก return ออกมาได้ และ ทำให้ transitive dependencies (dependency ตอนที่ modules require กันเอง) ถูกโหลดโดยไม่ทำให้เกิดลูป (cyclic dependency)

<!-- How is that helping for our injection? Well, if we export an object we are actually exporting a [singleton](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#singletonpatternjavascript) and every other module that imports the file will get the same object. This allows us to `register` our dependencies and later `fetch` them in another file. -->

แล้วสิ่งเหล่านี้จะช่วยเราในการทำ injection อย่างไร ? มันช่วยเราได้เพราะออบเจ็กต์ที่ถูก export ออกมานั้น จริงๆแล้วคือ [singleton](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#singletonpatternjavascript) และทุก module ที่ import ไฟล์นั้นเข้าไป ก็จะเข้าถึงอ็อบเจกต์ตัวเดียวกัน นั้นทำให้เราสามารถ ใส่ dependencies ของเราลงไป (`register`) และ นำออกมาจากไฟล์อื่นๆได้ (`fetch`)

<!-- Let's create a new file called `di.jsx` with the following content: -->

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

<!-- We'll store the dependencies in `dependencies` global variable (it's global for our module, not for the whole application). We then export two functions `register` and `fetch` that write and read entries. It looks a little bit like implementing setter and getter against a simple JavaScript object. Then we have the `wire` function that accepts our React component and returns a [higher-order component](https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components). In the constructor of that component we are resolving the dependencies and later while rendering the original component we pass them as props. We follow the same pattern where we describe what we need (`deps` argument) and extract the needed props with a `mapper` function.

Having the `di.jsx` helper we are again able to register our dependencies at the entry point of our application (`app.jsx`) and inject them wherever (`Title.jsx`) we need. -->

เราะจะเก็บ dependecies ไว้ในตัวแปร global ชื่อ `dependencies` (ตัวแปร global ในระดับ module ไม่ใช้ระดับแอปพลิเคชัน) หลังจากนั้นเราจะ export สองฟังก์ชันได้แก่ `register` และ `fetch` ที่จะทำหน้าที่เขียนและอ่านค่าต่างๆ โดยที่มันจะคล้ายๆกับการสร้าง setter และ getter ในออบเจ็กต์ของ Javascript ต่อจากนั้นเราจะใช้ฟังก์ชัน `wire` ในการรับ React component และ return [higher-order component](https://github.com/krasimir/react-in-patterns/tree/master/patterns/higher-order-components) ออกไป และใน constructor ของ component ที่อยู่ข้างในฟังก์ชัน wire เราจะทำการดึง dependencies ออกมา แล้วก็ส่งมันลงไปหา component ข้างใต้ที่กำลัง render ในรูปแบบของ props โดยที่เราจะทำตาม pattern เดิม ที่เราอธิบายสิ่งที่เราต้องการ (`deps` argument) และ ดึง props ที่ต้องการออกมาผ่านฟังก์ชัน `mapper`

การที่เรามี `di.jsx` helper นั้นทำให้เราสามารถสร้าง dependencies ได้ที่จุดเริ่มต้นของแอปพลิเคชัน (`app.jsx`) และ ส่งมันลงไปในที่ๆเราต้องการได้ (`Title.jsx`)

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

<!-- *If we look at the `Title.jsx` file we'll see that the actual component and the wiring may live in different files. That way the component and the mapper function become easily unit testable.* -->

*ถ้าเรามองที่ไฟล์ `Title.jsx` เราจะเห็นว่าตัว component และ ส่วนที่ทำการเชื่อมต่อนั้นสามารถอยู่คนละไฟล์ได้ ซึ่งท่านี้จะทำให้ตัว component และฟังก์ชัน mapper นั้นง่ายต่อการเทสในระดับ unit*

<!-- ## Final thoughts -->
## ความคิดทิ้งท้าย

<!-- Dependency injection is a tough problem. Especially in JavaScript. Lots of people didn't realize that but putting a proper dependency management is a key process of every development cycle. JavaScript ecosystem offers different tools and we as developers should pick the one that fits in our needs. -->

Dependency injection นั้นเป็นปัญหาที่ยากโดยเฉพาะใน Javascript หลายๆคนไม่คำนึงถึงว่าการทำ dependency management ที่เหมาะสมนั้น เป็นกระบวนการสำคัญในทุก development cycle และในส่วนของ JavaScript ecosystem นั้น มี tools ที่หลากหลายมานำเสนอให้เราอยู่เสมอ และ เรา developers ควรที่จะเลือกหยิบสิ่งที่ตอบโจทย์ต่อความต้องการของเรามากที่สุด
