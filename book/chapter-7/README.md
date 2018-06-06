# การไหลข้อมูลแบบทิศทางเดียว

การไหลข้อมูลแบบทิศทางเดียว (One-way direction data flow) คือรูปแบบที่ทำงานได้ดีกับ React มันเป็นไอเดียที่บอกว่าคอมโพเนนท์จะไม่แก้ไขข้อมูลใดๆที่ได้รับมา ซึ่งคอมโพเนนท์จะคอยดูการเปลี่ยนแปลงของข้อมูลเท่านั้น โดยอาจสร้างข้อมูลใหม่ แต่จะไม่อัพเดทข้อมูลที่ได้รับมา การเปลี่ยนแปลงของข้อมูล จะเกิดขึ้นตามกลไกจากอีกที่หนึ่ง และคอมโพเนนท์จะเป็นเพียงตัว render ด้วยข้อมูลชุดใหม่เท่านั้น

มาดูตัวอย่างง่ายๆของคอมโพเนนท์ `Switcher` กันเถอะ ซึ่งมันประกอบไปด้วยปุ่ม 1 ปุ่ม โดยถ้าคลิกปุ่มเราจะสามารถเปิดค่า flag บางอย่างในระบบได้

```js
class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { flag: false };
    this._onButtonClick = e => this.setState({
      flag: !this.state.flag
    });
  }
  render() {
    return (
      <button onClick={ this._onButtonClick }>
        { this.state.flag ? 'lights on' : 'lights off' }
      </button>
    );
  }
};

// ... render คอมโพเนนท์ที่นี่
function App() {
  return <Switcher />;
};
```

ในตอนนี้เรามีข้อมูลภายในคอมโพเนนท์ของเราแล้ว หรือพูดอีกอย่างหนึ่งได้ว่า ในคอมโพเนนท์ `Switcher` เป็นที่ๆเดียวที่รู้เกี่ยวกับค่า `flag` ของเรา ดังนั้นมาลองส่งมันออกไปยังตัวเก็บข้อมูล (`Store`) กันเถอะ 


```js
var Store = {
  _flag: false,
  set: function(value) {
    this._flag = value;
  },
  get: function() {
    return this._flag;
  }
};

class Switcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = { flag: false };
    this._onButtonClick = e => {
      this.setState({ flag: !this.state.flag }, () => {
        this.props.onChange(this.state.flag);
      });
    }
  }
  render() {
    return (
      <button onClick={ this._onButtonClick }>
        { this.state.flag ? 'lights on' : 'lights off' }
      </button>
    );
  }
};

function App() {
  return <Switcher onChange={ Store.set.bind(Store) } />;
};
```

ออบเจ็ค `Store` ของเราคือ [Singleton](https://addyosmani.com/resources/essentialjsdesignpatterns/book/#singletonpatternjavascript) ที่ๆเราสามารถมีตัวช่วยสำหรับการเขียนและอ่านข้อมูลของพร็อบเพอร์ตี้ `_flag` การส่งตัวเขียนข้อมูลไปยัง `Switcher` จะทำให้เราสามารถอัพเดทข้อมูลจากภายนอกได้ ดังนั้นภาพรวมของระบบการทำงานของแอพพลิเคชั่นจะมีลักษณะไม่ต่างไปจากนี้

![การไหลข้อมูลแบบทิศทางเดียว](./one-direction-1.jpg)

สมมติว่าเรากำลังบันทึกค่าของ flag ไปยังเซอร์วิสของระบบหลังบ้านผ่านทาง `Store` เมื่อผู้ใช้กลับมาใช้งานอีกครั้ง เราจะต้องบันทึกสถานะเริ่มต้น (Initial state) ให้ถูกต้อง ถ้าผู้ใช้ออกจากการใช้งานโดยมีค่า flag เป็น `true` เราต้องแสดง *"lights on"* แทนที่จะเป็นค่าเริ่มต้น *"lights off"* ตอนนี้การทำงานเริ่มจะยุ่งยากแล้วเพราะว่าเราต้องมีข้อมูลในสองที่ด้วยกันคือ ส่วนของการแสดงข้อมูล (UI) และส่วนของ `Store` ซึ่งแต่ละส่วนนั้นเก็บสถานะ (state) ของตัวเอง เราจึงต้องสื่อสารในสองทิศทางจาก store ไปยัง switcher และจาก switcher กลับไปยัง store

```js
// ... in App component
<Switcher
  value={ Store.get() }
  onChange={ Store.set.bind(Store) } />

// ... in Switcher component
constructor(props) {
  super(props);
  this.state = { flag: this.props.value };
  ...
```

รูปแบบการทำงานของเราจะเปลี่ยนไปเป็นลักษณะตามภาพดังนี้

![การไหลข้อมูลแบบสองทิศทาง](./one-direction-2.jpg)

จากรูปแบบของการทำงานทั้งหมดนี้ทำให้เราต้องจัดการสถานะสองที่แทนที่จะจัดการภายในที่เดียว ดังนั้นจะเกิดอะไรขึ้นถ้าค่าใน `Store` มีการเปลี่ยนแปลงโดยขึ้นอยู่กับการกระทำอื่นๆในระบบ เราต้องส่งการเปลี่ยนแปลงนั้นกลับมาที่ `Switcher` และเราก็เพิ่มความซับซ้อนของแอพ

การไหลข้อมูลแบบทิศทางเดียวสามารถแก้ปัญหานี้ได้ มันกำจัดการจัดการสถานะหลายๆที่ด้วยการจัดการสถานะเพียงที่เดียว ซึ่งปกติแล้วที่ๆนั้นมักจะเป็น store เพื่อที่จะทำแบบนั้นได้เราต้องปรับปรุงออบเจ็ค `Store` เล็กน้อย และเราจำเป็นต้องมีลอจิกที่สามารถให้เราติดตามการเปลี่ยนแปลงได้ด้วย

<span class="new-page"></span>

```js
var Store = {
  _handlers: [],
  _flag: '',
  subscribe: function(handler) {
    this._handlers.push(handler);
  },
  set: function(value) {
    this._flag = value;
    this._handlers.forEach(handler => handler(value))
  },
  get: function() {
    return this._flag;
  }
};
```

Then we will hook our main `App` component and we'll re-render it every time when the `Store` changes its value:

```js
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { value: Store.get() };
    Store.subscribe(value => this.setState({ value }));
  }
  render() {
    return (
      <div>
        <Switcher
          value={ this.state.value }
          onChange={ Store.set.bind(Store) } />
      </div>
    );
  }
};
```

Because of this change the `Switcher` becomes really simple. We don't need the internal state and the component may be written as a stateless function.

```js
function Switcher({ value, onChange }) {
  return (
    <button onClick={ e => onChange(!value) }>
      { value ? 'lights on' : 'lights off' }
    </button>
  );
};

<Switcher
  value={ Store.get() }
  onChange={ Store.set.bind(Store) } />
```

## Final thoughts

The benefit that comes with this pattern is that our components become dummy representation of the store's data. There is only one source of truth and this makes the development easier. If you are going to take one thing from this book I would prefer to be this chapter. The one-direction data flow drastically changed the way of how I think when designing a feature so I believe it will have the same effect on you.
