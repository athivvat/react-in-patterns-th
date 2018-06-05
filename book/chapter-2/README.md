# การติดต่อสื่อสาร

ทุกๆ React component เป็นเหมือนระบบเล็กๆที่บริหารจัดการตัวเอง ตัวมันเองจะมีข้อมูลสถานะ (state) เป็นของตัวเอง และมีส่วนของ ข้อมูลนำเข้า (input) และ ข้อมูลส่งออก (output) ซึ่งเป็นคุณลักษณะที่เราจะกล่าวถึงในหัวข้อนี้

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

ตัวอย่างข้างต้นยังได้มีการระบุ `proptypes` หรือชนิดของข้อมูลนำเข้า ซึ่งเป็นสิ่งที่ควรกำหนดให้ถูกต้องตามแต่ละชนิดของข้อมูลคุณลักษณะ เพื่อที่ React จะสามารถแจ้งเตือนเราได้หากมีการส่งชนิดข้อมูลที่ไม่ตรงกับที่ระบุไว้

`defaultProps` ก็เป็นอีกสิ่งหนึ่งที่มีประโยชน์ในการกำหนดค่าเริ่มต้นให้กับข้อมูลนำเข้า ซึ่งเราอาจจะต้องการกำหนดค่าเริ่มต้นที่เราต้องการเผื่อไว้ในกรณีที่ผู้เรียกใช้ component ของเราลืมส่งข้อมูลมาให้

React ไม่ได้กำหนดเจาะจงชนิดของข้อมูลนำเข้าของ component เลย มันอาจจะเป็นอะไรก็ได้ เราสามารถส่งแม้แต่ component อื่นๆ เข้ามาเป็นข้อมูลนำเข้าของ component ของเราอีกทีหนึ่ง ดังเช่นตัวอย่างด้านล่าง:

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

ในตัวอย่างข้างต้นนี้ ตรง `<span>community</span>` ใน parent component ที่ชื่อ `App` คือ ข้อมูลนำเข้าที่เป็น component ลูก (`children prop`) ใน component `Title` ท่านจะสังเกตุเห็นได้ว่าถ้าหากเราไม่มีบรรทัด `{ children }` ที่เป็นส่วนนึงของโค้ด `Title` จะทำให้ `<span>community</span>` ไม่ถูกแสดงผล

(ก่อนเวอร์ชั่น 16.3) มีข้อมูลนำเข้าทางอ้อมที่ส่งไปให้ component เรียกว่า `context` ซึ่ง component ทั้งหมดที่อยู่ภายใต้ลำดับชั้นของ context นั้นๆ สามารถที่จะเข้าถึงข้อมูล content นั้นได้ (จะกล่างถึงอย่างละเอียดอีกครั้งในหัวข้อ [dependency injection](../chapter-10/README.md) ) 

## Output

The first obvious output of a React component is the rendered HTML. Visually that is what we get. However, because the prop may be everything including a function we could also send out data or trigger a process.

In the following example we have a component that accepts the user's input and sends it out (`<NameField />`).

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

Very often we need an entry point of our logic. React comes with some handy lifecycle methods that may be used to trigger a process. For example we have an external resource that needs to be fetched on a specific page.

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

Let's say that we are building a search-results experience. We have a search page and we enter our criteria there. We click submit and the user goes to `/results` where we have to display the result of the search. Once we land on the results page we render some sort of a loading screen and trigger a request for fetching the results in `componentDidMount` lifecycle hook. When the data comes back we pass it to a `<List>` component.

## Final thoughts

It is nice that we may think about every React component as a black box. It has its own input, lifecycle and output. It is up to us to compose these boxes. And maybe that is one of the advantages that React offers. Easy to abstract and easy to compose.
