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
ผลลัพธ์ของโค้ดด้านบนจะได้อินพุตที่เราสามารถกำหนดค่าที่อินพุตนั้นแสดงอยู่ได้ แต่จะไม่สามารถเปลี่ยนแปลงค่าของมันได้เลย เพราะว่าเราได้กำหนดค่าให้อินพุตนั้นโดยนำมาจากค่า state ของ component `App`แต่ถ้าจะให้อินพุตนั้นใช้งานได้อย่างปกติอย่างที่ควรจะเป็น (คือสามารถกำหนดค่า และ เปลี่ยนแปลงค่าของมันได้) จำเป็นจะต้องเพิ่ม attribute handler (prop) ที่เรียกว่า `onChange` เพื่อทำการจัดการและเปลี่ยนค่า state ของ component `App`(ที่ถูกนำไปกำหนดเป็นค่าของอินพุต) ซึ่งจะทำให้เกิดวัฐจักรของการ render เกิดขึ้นใหม่แล้วจึงจะแสดงผลของค่าที่ได้อัพเดทไปแล้วที่อินพุต

The result of this code is an input element that we can focus but can't change. It is never updated because we have a single source of truth - the `App`'s component state. To make the input works as expected we have to add an `onChange` handler and update the state (the single source of truth). Which will trigger a new rendering cycle and we will see what we typed.

ในที่*อินพุตอิสระ*เป็นอินพุตที่ปล่อยให้เบราเซอร์เป็นตัวจัดการค่าต่างๆที่เกิดขึ้นมาจากการกระทำของยูสเซอร์ แต่ถึงอย่างนั้นเราก็ยังสามารถกำหนดค่าเริ่มต้นให้แก่อินพุตได้โดนการเพิ่ม attribute (prop) ที่เรียกว่า `defaultValue`แล้วหลังจากนั้นเบราเซอร์จะรับหน้าที่เก็บค่าของอินพุตและแสดงผลเอง

On the opposite side is the *uncontrolled* input where we let the browser handles the user's updates. We may still provide an initial value by using the `defaultValue` prop but after that the browser is responsible for keeping the state of the input.
