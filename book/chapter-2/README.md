# การติดต่อสื่อสาร

ทุก ๆ React component เป็นเหมือนระบบเล็ก ๆ ที่บริหารจัดการตัวเอง ตัวมันเองจะมีข้อมูลสถานะ (state) เป็นของตัวเอง และมีส่วนของ ข้อมูลนำเข้า (input) และ ข้อมูลส่งออก (output) ซึ่งเป็นคุณลักษณะที่เราจะกล่าวถึงในหัวข้อนี้

![Input-Output](./communication.jpg)

## ข้อมูลนำเข้า (Input)

ข้อมูลนำเข้าของ React component คือสิ่งที่เรียกว่า props (Properties หรือข้อมูลคุณลักษณะของมันนั่นเอง) ซึ่งก็คือสิ่งที่กำหนดว่าเราจะเราสามารถส่งข้อมูลอะไรเข้าไปที่ตัว component ได้บ้าง

```js
// Title.jsx
function Title(props) {
  return <h1>{ props.text }</h1>;
}
Title.propTypes = {
  text: PropTypes.string
};
Title.defaultProps = {
  text: 'Hello world'
};

// App.jsx
function App() {
  return <Title text='Hello React' />;
}
```

component ข้างต้นที่มีชื่อว่า `Title` และมีการรับข้อมูลนำเข้าเพียงข้อมูลเดียวคือ `text` ซึ่งควรจะถูกส่งมาจาก parent component (component ที่ห่อหุ้ม Title อีกทีหนึ่ง)

ตัวอย่างข้างต้นยังได้มีการระบุ `propTypes` หรือชนิดของข้อมูลนำเข้า ซึ่งเป็นสิ่งที่ควรกำหนดให้ถูกต้องตามแต่ละชนิดของข้อมูลคุณลักษณะ เพื่อที่ React จะสามารถแจ้งเตือนเราได้หากมีการส่งชนิดข้อมูลที่ไม่ตรงกับที่ระบุไว้

`defaultProps` ก็เป็นอีกสิ่งหนึ่งที่มีประโยชน์ในการกำหนดค่าเริ่มต้นให้กับข้อมูลนำเข้า ซึ่งเราอาจจะต้องการกำหนดค่าเริ่มต้นที่เราต้องการเผื่อไว้ในกรณีที่ผู้เรียกใช้ component ของเราลืมส่งข้อมูลมาให้

React ไม่ได้กำหนดเจาะจงชนิดของข้อมูลนำเข้าของ component เลย มันอาจจะเป็นอะไรก็ได้ เราสามารถส่งแม้แต่ component อื่น ๆ เข้ามาเป็นข้อมูลนำเข้าของ component ของเราอีกทีหนึ่ง ดังเช่นตัวอย่างด้านล่าง:

```js
function SomethingElse({ answer }) {
  return <div>The answer is { answer }</div>;
}
function Answer() {
  return <span>42</span>;
}

// later somewhere in our application
<SomethingElse answer={ <Answer /> } />
```

สังเกตุตรง `props.children` ซึ่งถือเป็นข้อมูลนำเข้าอีกชนิดที่ทำให้เราสามารถที่จะเข้าถึง component ลูก (child components) ที่ถูกระบุอยู่ใน component ที่เรียกใช้ component ของเราอีกที 
ดังเช่นตัวอย่างด้านล่าง:

```js
function Title({ text, children }) {
  return (
    <h1>
      { text }
      { children }
    </h1>
  );
}
function App() {
  return (
    <Title text='Hello React'>
      <span>community</span>
    </Title>
  );
}
```

ในตัวอย่างข้างต้นนี้ ตรง `<span>community</span>` ที่อยู่ใน `App` component เป็น component ลูก (`children`) ของ component `Title` ท่านจะสังเกตุเห็นได้ว่าถ้าหากเราไม่มีบรรทัด `{ children }` ที่เป็นส่วนนึงของโค้ด `Title` จะทำให้ `<span>community</span>` ไม่ถูกแสดงผล

(ก่อนเวอร์ชั่น 16.3) มีข้อมูลนำเข้าทางอ้อมที่ส่งไปให้ component เรียกว่า `context` ซึ่ง component ทั้งหมดที่อยู่ภายใต้ลำดับชั้นของ context นั้น ๆ สามารถที่จะเข้าถึงข้อมูล context นั้นได้ (จะกล่างถึงอย่างละเอียดอีกครั้งในหัวข้อ [dependency injection](../chapter-10/README.md) ) 

## ข้อมูลส่งออก (Output)

ข้อมูลส่งออกแบบแรกที่ชัดเจนที่สุดของ React component ก็คือ HTML ที่ถูกประมวลผลออกมาแล้ว เป็นสิ่งที่สามารถเห็นได้ง่าย ๆ อย่างไรก็ตาม เพราะว่าเราสามารถที่จะส่งอะไรเข้ามาเป็นข้อมูลนำเข้าก็ได้ เราจึงสามารถที่จะส่งฟังก์ชัน (function) เข้ามาเพื่อที่จะส่งข้อมูลกลับออกไปหรือกระตุ้นให้เกิดการเริ่มกระบวนการที่ต้องการได้ด้วย

ในตัวอย่างด้านล่างเรามี component ชื่อ `<NameField />` ที่ด้านในของมันเป็น html input tag ทำหน้าที่รับข้อมูลนำเข้าจากผู้ใช้และมี prop (ข้อมูลนำเข้าของตัว NameField เอง) ที่ชื่อว่า `valueUpdated` เป็นเหมือน callback (ฟังก์ชันที่ส่งข้อมูลกลับเมื่อสิ้นสุดการทำงาน) ที่คอยส่งค่าที่ user ป้อนเข้ามา ออกไปยัง component ที่เรียกใช้ตัว `<NameField />` อีกทีนึง (`<App />`)

<span class="new-page"></span>

```js
function NameField({ valueUpdated }) {
  return (
    <input
      onChange={event => valueUpdated(event.target.value) } />
  );
};
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { name: '' };
  }
  render() {
    return (
      <div>
        <NameField
          valueUpdated={ name => this.setState({ name }) } />
        Name: { this.state.name }
      </div>
    );
  }
};
```

บ่อยครั้งที่เราต้องการจุดเริ่มต้นสำหรับ logic ของเรา และ React มาพร้อมกับ lifecycle method ต่าง ๆ ที่เราสามารถใช้ในการระบุการทำงานที่ต้องการในแต่ละช่วงสถานะของตัว component ได้ ดังเช่นตัวอย่างด้านล่างที่เราพยายามจะดึงข้อมูลจากทรัพยากรภายนอก

```js
class ResultsPage extends React.Component {
  componentDidMount() {
    this.props.getResults();
  }
  render() {
    if (this.props.results) {
      return <List results={ this.props.results } />;
    } else {
      return <LoadingScreen />
    }
  }
}
```

ลองคิดว่าเรากำลังจะสร้างส่วนของการค้นหา ซึ่งเรามีหน้าสำหรับค้นหาที่รับเงื่อนไขในการค้นหาอยู่แล้ว user อาจจะกรอกเงื่อนไขและกดค้นหา ซึ่งจะนำผู้ใช้ไปอยู่หน้า `/results` ที่ ๆ เราจะแสดงผลของการค้นหาของเรา และเมื่อผู้ใช้เข้าสู่หน้าแสดงผลสำเร็จแล้วเราก็จะให้ผู้ใช้พบกับส่วนที่แสดงว่ากำลังทำการดึงข้อมูลอยู่ให้ผู้ใช้รอ พลางทำการร้องขอข้อมูลไปที่ทรัพยากรด้านนอก ในขั้นตอนนี้เราจะทำใน `componentDidMount` ที่เป็น lifecycle method ของ React component และเมื่อเราได้ผลลัพธ์กลับมาจากแหล่งข้อมูลที่เราร้องขอ เราก็จะนำข้อมูลมาแสดงให้กับผู้ใช้ใน `<List>` component ตามโค้ดตัวอย่างด้านบน

## ข้อคิด

มันอาจจะง่ายขึ้นหากว่าเราคิดว่าทุก ๆ React component เป็นเหมือนกล่องดำ (black box) ที่มันจะมีรูปแบบของข้อมูลนำเข้า ข้อมูลส่งออก และ lifecycle ของมันเอง ขึ้นอยู่กับเราว่าจะประกอบกล่องต่าง ๆ เหล่านี้อย่างไร และบางทีนี่อาจจะเป็นหนึ่งในข้อได้เปรียบที่ React มีให้กับเรา ซึ่งก็คือการที่มันง่ายต่อการนิยามและง่ายต่อการประกอบขึ้นมาให้เป็นสิ่่งที่ใช้ได้จริง ๆ
