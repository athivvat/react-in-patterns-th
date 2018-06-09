# อินพุตควบคุม (Controlled Input) และ อินพุตอิสระ (Uncontrolled Input)

*อินพุตควบคุม (Controlled Input)* และ *อินพุตอิสระ (Uncontrolled Input)* จะถูกนำไปใช้ในการจัดการข้อมูล หรือ action ต่างๆของ form

*อินพุตควบคุม (Controlled Input)* นั้นค่าของอินพุตจะถูกกำหนดด้วยข้อมูลจากภาคนอก ที่เรามักจะใช้ค่านี้จากแหล่งข้อมูลที่หนึ่งที่มีค่าความจริงเพียงหนึ่งเดียวเท่านั้น (Single source of thruth) ดังตัวอย่างด้านล่าง component `App` มี element `<input>` อยู่หนึ่งตัว ซึ่งเป็น *Controlled Input* 

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

ในขณะที่ *อินพุตอิสระ (Uncontrolled Input)* เป็นอินพุตที่ปล่อยให้เบราเซอร์เป็นตัวจัดการค่าต่างๆที่เกิดขึ้นมาจากการกระทำของยูสเซอร์ แต่ถึงอย่างนั้นเราก็ยังสามารถกำหนดค่าเริ่มต้นให้แก่อินพุตได้โดนการเพิ่ม attribute (prop) ที่เรียกว่า `defaultValue`แล้วหลังจากนั้นเบราเซอร์จะรับหน้าที่เก็บค่าของอินพุตและแสดงผลเอง

จากตัวอย่างข้างบนนั้น element `<input>` ค่อนข้างจะไม่มีประโยชน์ เนื่องจากเมื่อมีการอัพเดทข้อมูลของยูสเซอร์ ตัว component `App` นั้นจะไม่รับรู้อะไรเลย จะต้องใช้ตัวอ้างอิง [`Refs`](https://reactjs.org/docs/glossary.html#refs) เพื่อที่จะดึงข้อมูลจากอินพุตโดยตรง

prop `ref` นั้นจะรับตัวอักษรสตริง หรือ callback function จากตัวอย่างซอสโค้ดด้านบนใช้ callback เพื่อที่จะเก็บ DOM element ไว้ที่ตัวแปร *local* ที่มีชื่อว่า `input`ภายหลังเมื่อใช้ handler `onChange` `App`'s state.

*Using a lot of `refs` is not a good idea. If it happens in your app consider using `controlled` inputs and re-think your components.*

## Final thoughts

*controlled* versus *uncontrolled* inputs is very often underrated. However I believe that it is a fundamental decision because it dictates the data flow in the React component. I personally think that *uncontrolled* inputs are kind of an anti-pattern and I'm trying to avoid them when possible.


