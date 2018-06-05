# การจัดการอีเวนท์

React นั้นได้มีการเตรียม attributes ต่างๆที่ใช้สำหรับการจัดการกับ `event` ไว้เรียบร้อยแล้ว ซึ่งวิธีใช้ทั่วไปนั้นแทบจะเหมือนกับวิธีการจัดการ `event` ใน DOM ที่เราคุ้นเคยเลย โดยจะมีความแตกต่างกันเพียงเล็กน้อย เช่น การใช้ `camelCase` เป็นชื่อ attribute หรือการส่ง *Function* แทนที่จะเป็น *String* เป็นต้น

```js
const theLogoIsClicked = () => alert('Clicked');

// อีเวนท์ onClick
<Logo onClick={ theLogoIsClicked } />

// อีเวนท์ onChange
<input type='text' onChange={event => theInputIsChanged(event.target.value) } />
```

ส่วนใหญ่แล้วเรามักจะจัดการอีเวนท์กันภายใน Component ที่สร้างอีเวนท์นั้นขึ้นมา เช่นในตัวอย่างข้างล่าง เรามี `button` อยู่ในคอมโพเนนท์ `Switcher` แล้วเราต้องการให้การคลิกที่ `button` ไปรันคำสั่งชื่อ `_handleButtonClick` ที่อยู่ในคอมโพเนนท์ `Switcher`

```js
class Switcher extends React.Component {
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log('Button is clicked');
  }
};
```

โค้ดชุดจะสามารถทำงานได้ตรงตามที่เราต้องการ เพราะ `_handleButtonClick` นั้นเป็น *Function* และเราก็ส่ง *Function* เข้าไปใน attribute ชื่อ `onClick`.

That's all fine because `_handleButtonClick` is a function and we indeed pass a function to the `onClick` attribute. The problem is that as it is the code doesn't keep the same context. So, if we have to use `this` inside `_handleButtonClick` to refer the current `Switcher` component we will get an error.

```js
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
  }
  render() {
    return (
      <button onClick={ this._handleButtonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
    // leads to
    // Uncaught TypeError: Cannot read property 'state' of null
  }
};
```

What we normally do is to use `bind`:

```js
<button onClick={ this._handleButtonClick.bind(this) }>
  click me
</button>
```

However, this means that the `bind` function is called again and again because we may render the button many times. A better approach would be to create the bindings in the constructor of the component:

<span class="new-page"></span>

```js
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
    this._buttonClick = this._handleButtonClick.bind(this);
  }
  render() {
    return (
      <button onClick={ this._buttonClick }>
        click me
      </button>
    );
  }
  _handleButtonClick() {
    console.log(`Button is clicked inside ${ this.state.name }`);
  }
};
```

Facebook by the way [recommend](https://facebook.github.io/react/docs/reusable-components.html#no-autobinding) the same technique while dealing with functions that need the context of the same component.

The constructor is also a nice place for partially executing our handlers. For example, we have a form but want to handle every input in a single function.

<span class="new-page"></span>

```js
class Form extends React.Component {
  constructor(props) {
    super(props);
    this._onNameChanged = this._onFieldChange.bind(this, 'name');
    this._onPasswordChanged =
      this._onFieldChange.bind(this, 'password');
  }
  render() {
    return (
      <form>
        <input onChange={ this._onNameChanged } />
        <input onChange={ this._onPasswordChanged } />
      </form>
    );
  }
  _onFieldChange(field, event) {
    console.log(`${ field } changed to ${ event.target.value }`);
  }
};
```

## Final thoughts

There is not much to learn about event handling in React. The authors of the library did a good job in keeping what's already there. Since we are using HTML-like syntax it makes total sense that we have also a DOM-like event handling.
