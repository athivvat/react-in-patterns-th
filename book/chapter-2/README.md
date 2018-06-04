# การสื่อสาร

คอมโพเน้นท์ (React Component) เหมือนกับระบบเล็กๆที่สามารถจัดการหน้าที่ของมันได้ด้วยตัวมันเอง ซึ่งจะประกอบด้วย State, Input, และ Output เนื้อหาต่อไปเราจะมาดูคอมโพเน้นท์อย่างละเอียดกัน

![Input-Output](./communication.jpg)

## Input

ใช้ props ในการส่งผ่านข้อมูลเข้าไปในคอมโพเน้นท์

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

คอมโพเน้นท์ `Title` มีข้อมูลที่ส่งผ่านเข้ามา 1 ตัวคือ `text` (prop) ในคอมโพเน้นท์แม่ (`App`) ต้องส่ง prop ตัวนี้เข้ามาผ่านแอทริบิวท์ของ <Title> สิ่งที่สำคัญอย่างยิ่งเราต้องไม่ลืมที่จะกำหนดรูปแบบของ prop ที่ส่งเข้ามาในคอมโพเน้นท์ด้วย `propTypes` เพื่อที่จะป้องการการส่งรูปแบบของ prop ที่ผิดไปจากรูปแบบที่เราต้องการ ซึ่งในตัวอย่างเรากำหนดรูปแบบของ text เป็น string ถ้าไม่ได้ส่งมาเป็น string จะมีข้อความเตือนในคอนโซล อีกหนึ่งอย่างที่ไม่ควรลืมก็คือ `defaultProps` ซึ่งจะช่วยตั้งค่าตั้งต้นของ prop นั้นๆให้ เพื่อป้องการการลืมใส่ prop ที่จำเป็นต่อคอมโพเน้นท์ของนักพัฒนาโปรแกรม

React ไม่มีกฏเกณฑ์ในการกำหนดการส่ง prop ซึ่งสามารถเป็นคอมโพเน้นท์ หรืออื่นๆ ตามที่ต้องการส่ง

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

`props.children` สามารถที่จะเข้าถึงคอมโพเน้นทลูกที่ถูกส่งผ่านมาจากคอมโพเน้นท์แม่ได้ ยกตัวอย่างเช่น:

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

จากตัวอย่างข้างต้น `<span>community</span>` ในคอมโพเน้นท์ `App` คือ `children` ในคอมโพเน้นท์ `Title` สังเกตว่า ถ้าเราไม่ใส่ `{ children }` ในส่วนรีเทิร์นของ `Title` แท็ก `<span>` จะไม่โชว์ค่าออกมา

(ก่อนเวอร์ชั่น 16.3) เราสามารถรับค่าโดยไม่ผ่าน prop มาในคอมโพเน้นท์ได้ด้วย `context` ซึ่งเราจะสามารถเข้าถึง `context` ได้ในทุกๆคอมโพเน้นท์ใน React และเราจะพูดถึง `context` อย่างละเอียดใน [dependency injection](../chapter-10/README.md)

## Output

Output ของคอมโพเน้นท์คือ HTML อย่างไรก็ตามเนื่องจากว่า prop อาจจะเป็นทุกสิ่งอย่างรวมถึงฟังก์ชั่น เราสามารถที่จะส่งข้อมูล หรือเรียกฟังก์ชั่นการทำงานต่างๆได้ด้วย

ในตัวอย่างถัดไป เราจะมีคอมโพเน้นท์ที่รับข้อมูลผู้ใช้ และส่งออกไป (`<NameField />`)

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

React มีสิ่งที่เรียกว่า lifecycle method ไว้เรียกฟังก์ชั่นการทำงานต่างๆ ยกตัวอย่างเช่น ถ้าเราอยากจะดึงข้อมูลจาก Api เราสามารถเรียกได้ผ่าน lifecycle method ได้

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

จากตัวอย่างข้างบน สมมติว่าเราต้องเราที่จะสร้างเว็บที่ไว้ค้นหาข้อมูล เราจะต้องมีหน้าค้นหา จากนั้นเราก็กรอกข้อมูลเข้าไป และกดคลิก submit ผู้ใช้งานจะถูกส่งไปยังหน้า `/results` ซึ่งเป็นหน้าที่จะโชว์ข้อมูลที่ผู้ใช้ค้นหาเข้ามา เมื่อผู้ใช้งานเข้ามาถึงเราจะเรนเดอร์หน้าจอโหลด
และในเวลาเดียวกันเราจะโหลดข้อมูลผลการค้นหาที่เกี่ยวข้องผ่าน `componentDidMount` เมื่อโหลดข้อมูลเสร็จเราจะส่งข้อมูลให้คอมโพเน้นท์ `<List>` นำไปเรนเดอร์ต่อไป

## ความคิดเห็นสุดท้าย

มันเป็นเรื่องที่ดีที่ เราจะเปรียบเทียบคอมโพเน้นท์เป็นเหมือนกล่องดำ มันมี input, lifecycle, และ output ของตัวมันเอง ซึ่งมันอยู่ที่เราว่าเราจะให้กล่องดำนี้เป็นอย่างไร ซึ่งจุดนี้เป็นจุดที่โดดเด่นมากของ React
