# อินพุตควบคุม (Controlled Input) และ อินพุตอิสระ (Uncontrolled Input)

*Controlled Input* และ *Uncontrolled Input* จะถูกนำไปใช้ในการจัดการข้อมูล หรือ action ต่างๆของ form

*Controlled Input* นั้นค่าของ input จะถูกกำหนดด้วยข้อมูลจากภาคนอก ที่เรามักจะใช้ค่านี้จากแหล่งข้อมูลที่หนึ่งที่มีค่าความจริงเพียงหนึ่งเดียวเท่านั้น (Single source of thruth) ดังตัวอย่างด้านล่าง component `App` มี element `<input>` อยู่หนึ่งตัว ซึ่งเป็น *Controlled Input* 

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
