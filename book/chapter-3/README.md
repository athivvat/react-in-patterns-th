# การจัดการ Event

React นั้นได้มีการเตรียม attributes ต่าง ๆ ที่ใช้สำหรับการจัดการกับ `event` ไว้เรียบร้อยแล้ว ซึ่งวิธีใช้ทั่วไปนั้นแทบจะเหมือนกับวิธีการจัดการ `event` ใน DOM ที่เราคุ้นเคยเลย โดยจะมีความแตกต่างกันเพียงเล็กน้อย เช่น การใช้ `camelCase` เป็นชื่อ attribute หรือการส่ง function แทนที่จะเป็น string เป็นต้น

```js
const theLogoIsClicked = () => alert('Clicked');

// อีเวนท์ onClick
<Logo onClick={ theLogoIsClicked } />

// อีเวนท์ onChange
<input
  type='text'
  onChange={event => theInputIsChanged(event.target.value) } />
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

โค้ดชุดจะสามารถทำงานได้ตรงตามที่เราต้องการ เพราะ `_handleButtonClick` นั้นเป็น *Function* และเราก็ส่ง *Function* เข้าไปใน attribute ชื่อ `onClick`

**แต่!!** เนื่องจากตัวโค้ดไม่ได้อยู่ใน `context` (บริบท) เดียวกัน ส่งผลให้เวลาที่เราต้องการเรียกถึงตัวแปร `this` ข้างในฟังก์ชัน `_handleButtonClick` เพื่อเรียกถึงคอมโพเนนท์ `Switcher` จะทำให้เกิด Error ขึ้นมาทันที

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
    // ไม่สามารถเรียก this.state.name ได้ เพราะหา this ไม่เจอ เนื่องจาก context ของ this ไม่ตรงกัน
    // Uncaught TypeError: Cannot read property 'state' of null
  }
};
```

เราสามารถแก้ได้โดยการใช้ `bind`

```js
<button onClick={ this._handleButtonClick.bind(this) }>
  click me
</button>
```

เสียแต่ว่าฟังก์ชัน `bind` ของเรานั้นจะถูกเรียกซ้ำไปซ้ำมาอยู่บ่อยๆ เพราะว่าคอมโพเนนท์ `button` อาจถูก render ใหม่หลายๆครั้ง (หรือที่เราเรียกกันว่า re-render) วิธีที่ดีกว่านี้ก็คือการเปลี่ยนไป `bind` ที่ `constructor` ของคอมโพเนนท์นั้นทีเดียวเลย

<span class="new-page"></span>

```js
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { name: 'React in patterns' };
    // ทำการ binding ที่นี่แทน
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

Facebook (ผู้สร้าง React) เองก็ยัง [แนะนำ](https://reactjs.org/docs/handling-events.html) เทคนิคเดียวกันนี้เวลาที่ต้องจัดการกับฟังก์ชันที่ใช้ context เดียวกันภายในคอมโพเนนท์

Constructor ยังถือเป็นที่ที่ดีสำหรับการสร้าง handler ที่มีค่าบางอย่างพร้อมแล้วอีกด้วย ยกตัวอย่างเช่น เมื่อเรามี `<form>` ที่มีหลาย `<input>` อยู่ข้างใน แต่เราต้องการจัดการการทำงานเมื่อ `<input>` ถูกเปลี่ยนในฟังก์ชัน `_onFieldChange(field, event)` เพียงที่เดียว

<span class="new-page"></span>

```js
class Form extends React.Component {
  constructor(props) {
    super(props);
    this._onNameChanged = this._onFieldChange.bind(this, 'name');
    this._onPasswordChanged = this._onFieldChange.bind(this, 'password');
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

## สรุป

การจัดการอีเวนท์ใน React นั้นอาจดูเหมือนไม่มีอะไรใหม่ให้ศึกษาสักเท่าไหร่ เพราะคนสร้าง React นั้นถือว่าทำไว้ดีแล้วในเรื่องของการนำสิ่งที่มีอยู่แล้วมาใช้ ในเมื่อตัวไลบรารี่เองมีการใช้ syntax ที่เหมือนกับ HTML เดิมอยู่แล้ว จึงไม่ใช่เรื่องแปลกอะไรที่จะมีการจัดการอีเวนท์เหมือนใน DOM
