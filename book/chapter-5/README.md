# อินพุตควบคุม (Controlled Input) และ อินพุตอิสระ (Uncontrolled Input)

*อินพุตควบคุม (Controlled Input)* และ *อินพุตอิสระ (Uncontrolled Input)* จะถูกนำไปใช้ในการจัดการข้อมูล หรือ action ต่างๆของ form

*อินพุตควบคุม (Controlled Input)* นั้นค่าของอินพุตจะถูกกำหนดด้วยข้อมูลจากภายนอก ที่เรามักจะใช้ค่านี้จากแหล่งข้อมูลเดีนว (Single source of truth) ดังตัวอย่างด้านล่าง component `App` มี element `<input>` อยู่หนึ่งตัว ซึ่งเป็น *Controlled Input* 

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
  render() {
    return <input type='text' value={ this.state.value } />;
  }
};

```
ผลลัพธ์ของโค้ดด้านบนจะได้ input ที่เราสามารถกำหนดค่าที่ input นั้นแสดงอยู่ได้ แต่จะไม่สามารถเปลี่ยนแปลงค่าของมันได้เลย เพราะว่าเราได้กำหนดค่าให้อินพุตนั้นโดยนำมาจากค่า state ของ component `App` แต่ถ้าจะให้ input นั้นใช้งานได้อย่างที่ปกติมันควรจะเป็น (คือสามารถกำหนดค่า และ เปลี่ยนแปลงค่าของมันได้) จำเป็นจะต้องเพิ่ม handler ที่เรียกว่า `onChange` เพื่อทำการจัดการและเปลี่ยนค่า state ของ component `App` (ที่ถูกนำไปกำหนดเป็นค่าของอินพุต) ซึ่งทำให้เกิดการ render ใหม่แล้วจึงจะแสดงผลของค่าที่ได้อัพเดทไปแล้วที่ input
<span class="new-page"></span>

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
    this._change = this._handleInputChange.bind(this);
  }
  render() {
    return (
      <input
        type='text'
        value={ this.state.value }
        onChange={ this._change } />
    );
  }
  _handleInputChange(e) {
    this.setState({ value: e.target.value });
  }
};
```

ในขณะที่ *อินพุตอิสระ (Uncontrolled Input)* เป็น input ที่ปล่อยให้ browser เป็นตัวจัดการค่าต่างๆที่เกิดขึ้นมาจากการกระทำของ user แต่ถึงอย่างนั้นเราก็ยังสามารถกำหนดค่าเริ่มต้นให้แก่ input ได้โดยการเพิ่ม attribute (prop) ที่เรียกว่า `defaultValue` แล้วหลังจากนั้น browser จะรับหน้าที่เก็บค่าของ input และแสดงผลเอง

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
  }
  render() {
    return <input type='text' defaultValue={ this.state.value } />
  }
};
```

จากตัวอย่างข้างบนนั้น element `<input>` ค่อนข้างจะไร้ประโยชน์ เพราะถ้า user อัพเดทค่าของ input ตัว component `App` นั้นจะไม่รับรู้อะไรเลย จะต้องใช้ตัวอ้างอิง [`Refs`](https://reactjs.org/docs/glossary.html#refs) เพื่อที่จะดึงข้อมูลจาก input โดยตรง

```js
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = { value: 'hello' };
    this._change = this._handleInputChange.bind(this);
  }
  render() {
    return (
      <input
        type='text'
        defaultValue={ this.state.value }
        onChange={ this._change }
        ref={ input => this.input = input }/>
    );
  }
  _handleInputChange() {
    this.setState({ value: this.input.value });
  }
};
```

การจะใช้ Refs นั้นต้องกำหนด prop ที่ชื่อว่า `ref` และค่าที่กำหนดให้นั้นจะต้องเป็นตัวอักษรสตริง (Legacy String Refs) หรือ callback function* จากตัวอย่าง source code ด้านบนใช้ callback เพื่อที่จะเก็บ DOM element ไว้ที่ตัวแปร *local* ที่มีชื่อว่า `input` และเมื่อ handler `onChange` จับได้ว่า input มีการอัพเดท function ที่มาทำหน้าที่เป็น handler (ในที่นี้คือ `_handleInputChange()`) ก็จะใช้ Refs เพื่ออ้างถึงข้อมูลที่ DOM input นั้นถืออยู่ และนำไปใช้อัพเดทค่า state ของ component `App`

*ปัจจุบัน React สนับสนุนให้ใช้ callback function มากกว่า [Legacy String Refs](https://reactjs.org/docs/refs-and-the-dom.html#legacy-api-string-refs) เพราะแบบเก่ายังมี issue และอาจจะถูกนำออกไปในเวอร์ชั่นข้างหน้า
*การใช้  `Refs` บ่อยๆนั้นไม่ใช่ตัวเลือกที่ดีนัก ถ้าเป็นไปได้ควรใช้ หรือ migrate มาใช้ `อินพุตควบคุม` แทน*

## ข้อคิด

คนส่วนใหญ่มักจะมองข้าม ข้อแตกต่างระหว่าง *อินพุตควบคุม* และ *อินพุตอิสระ* แต่โดยพื้นฐานและแนวคิดของ React นั้นจะเป็นการควบคุม data flow เพราะฉะนั้นแนวคิดนี้ค่อนข้างจะสนับสนุนและสอดคล้องกับวิธีและกลไกของ *อินพุตควบคุม*
ส่วนตัวผมนั้นคิดว่าการใช้ *อินพุตอิสระ* ค่อนข้างจะเป็น anti-pattern ถ้าเป็นไปได้ผมมักจะพยายามหลีกเลี่ยงที่จะใช้มัน
