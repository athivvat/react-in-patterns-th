# การใช้งานคู่กับ Third-party Software

React อาจจะเป็นหนึ่งในตัวเลือกที่ดีทีสุดสำหรับการสร้างส่วนประสานกับผู้ใช้ (UI) ตัว React มีการออกแบบโครงสร้างที่ดี รวมถึงมีการสนับสนุน และมีกลุ่ม community ที่ดี อย่างไรก็ตามก็ยังมีในหลายกรณีที่เราต้องการจะใช้ service ภายนอก หรือต้องการที่จะใช้งานกับอะไรสักอย่างที่มันแตกต่างไปอย่างสิ้นเชิง พวกเราทั้งหมดรู้ว่า React ทำงานอย่างหนักกับตัว DOM (Document Object Model) จริง ๆ ของเว็บ และการควบคุมอะไรก็ตามที่จะแสดงผลออกมาทางหน้าจอเป็นพื้นฐาน นั่นก็คือเหตุผลที่การการใช้งานควบคู่กับ third-party software ค่อนข้างที่จะต้องใช้เทคนิคที่อาจจะยุ่งยาก ในส่วนนี้เราจะแสดงการใช้งาน React ควบคู่กับ jQuery's UI plugin และค่อย ๆ ทำมันอย่างปลอดภัย

## ตัวอย่าง

ผมได้เลือกใช้ [*tag-it*](https://github.com/aehlke/tag-it) ซิ่งเป็นส่วนเสริมตัวนึงของ jQuery มาใช้เป็นตัวอย่างนะครับ มันเอาไว้แปลงแท็ก (tag) ul ที่เอาไว้แสดงผลข้อมูลรายการที่ไม่เป็นลำดับให้กลายเป็นตัวป้อนข้อมูลที่จะไว้ใช้ในการจัดการแท็ก

จากภาษา HTML ด้านล่างนี้:

```html
<ul>
  <li>JavaScript</li>
  <li>CSS</li>
</ul>
```

แสดงผลได้ดังนี้:

![tag-it](./tag-it.png)

เพื่อให้มันทำงานได้ เราจำเป็นจะต้องมี jQuery, jQuery UI และที่ขาดไม่ได้คือ plugin *tag-it*; tag-it มันใช้งานประมาณนี้ครับ:

```jsx
$('<dom element selector>').tagit();
```

อธิบายก็คือเราเลือก DOM element และไปเรียกใช้งานฟังก์ชันที่ชื่อ `target()`

เอาละครับ มาสร้าง React app ง่าย ๆ ขึ้นมาตัวนึง ที่จะมาลองใช้กับ plugin:

```jsx
// Tags.jsx
class Tags extends React.Component {
  render() {
    return (
      <ul>
      { 
        this.props.tags.map(
          (tag, i) => <li key={ i }>{ tag } </li>
        )
      }
      </ul>
    );
  }
};

// App.jsx
class App extends React.Component {
  constructor(props) {
    super(props);

    this.state = { tags: ['JavaScript', 'CSS' ] };
  }
  render() {
    return (
      <div>
        <Tags tags={ this.state.tags } />
      </div>
    );
  }
}

ReactDOM.render(<App />, document.querySelector('#container'));
```

เข้าไปที่ class ที่ชื่อว่า `App` มันใช้งานตัว component ที่ชื่อว่า `Tags` ที่จะทำการแสดงผลเจ้าตัวรายการที่ไม่เป็นลำดับ (unordered list) ด้วยการส่งค่าผ่าน props ที่ชื่อว่า `tags` แล้วเมื่อ React ทำการแสดงรายการที่ว่าบนหน้าจอ เราจะรู้ว่าเรามีแท็ก `<ul>` เพื่อเราสามารถเชื่อมมันเข้ากับ jQuery plugin

## บังคับให้เกิดการ render เพียงแค่ครั้งเดียว (single-render)

สิ่งแรกที่เราจะต้องทำคือการบังคับให้เกิดการ render เพียงแค่ครั้งเดียว (single-render) ของ component `Tags` นั่นเพราะเมื่อ React เพิ่ม element ต่าง ๆ เข้าไปที่ DOM จริง ๆ (actual DOM) เราต้องการที่จะส่งการควบคุม element ต่าง ๆ ไปให้ jQuery ถ้าเราข้ามขั้นตอนนี้ไป React และ jQuery จะทำงานอยู่บน DOM ตัวเดียวกัน โดยไม่รู้ซึ่งกันและกัน เพื่อให้เกิดการ render ครั้งเดียว เราจะต้องใช้ method ที่อยู่ใน lifecycle ของ React ที่ชื่อว่า `shouldComponentUpdate` อย่างเช่นโค้ดด้านล่างนี้: 

```jsx
class Tags extends React.Component {
  shouldComponentUpdate() {
    return false;
  }
  ...
```


โดยการส่งค่ากลับมาเป็น `false` เสมออย่างนี้ เรากล่าวว่า component ของเราจะไม่ re-render อีก การใช้งานเมธอด `shouldComponentUpdate` เพื่อให้ React รู้ว่าต้องมีการ `render` ใหม่หรือไม่ นั่นเป็นสิ่งที่เราต้องทำเพราะว่า เราจะใช้ React เพื่อวางโครงสร้างเว็บ แต่หลังจากนั้นเราไม่ต้องการที่จะให้มันมายุ่งเกี่ยวกับการ `render` ใหม่อีก

## การเตรียมพร้อมสำหรับส่วนขยาย

React มี [API](https://facebook.github.io/react/docs/refs-and-the-dom.html) มาตัวนึงสำหรับการเข้าถึง DOM nodes จริงๆ ที่อยู่ใน HTML (actual DOM nodes) เราจะต้องใช้ attribute ที่ชื่อว่า `ref` กับตัว node และเราจะใช้การเข้าถึงตัว node ผ่าน `this.refs` ซึ่ง `componentDidMount` เป็น lifecycle method ที่เหมาะสำหรับการเรียกใช้ plugin *tag-it* นั่นเป็นเพราะว่ามันจะถูกเรียกเมื่อ React นำผลลัพธ์จาก method render ไป mount ใน DOM nodes จริง ๆ

<br /><br /><br />

```jsx
class Tags extends React.Component {
  ...
  componentDidMount() {
    this.list = $(this.refs.list);
    this.list.tagit();
  }
  render() {
    return (
      <ul ref='list'>
      { 
        this.props.tags.map(
          (tag, i) => <li key={ i }>{ tag } </li>
        )
      }
      </ul>
    );
  }
  ...
```

ตัวโค้ดที่อยู่ด้านบนกับเมธอด `shouldComponentUpdate` ทำให้ React จะ render ตัว `<ul>`  ที่มีสองอัน (<ul> และ </ul>) หลังจากนั้นตัว *tag-it* จะแปลงมันให้เป็น widget สำหรับการแก้ไขตัว tag

## การควบคุมส่วนขยายด้วย React

สมมติว่าเราต้องการที่จะโปรแกรมเพิ่ม แท็กตัวใหม่ที่กำลังทำงานอยู่แล้วกับ *tag-it* การทำงานดังกล่าวจะถูกเรียกใช้งานโดย React component และต้องการใช้ jQuery API ด้วย; เราต้องหาวิธีที่จะติดต่อ component ที่ชื่อว่า `Tags` ให้มีการ `render` ถ้ามีการแก้ไขข้อมูล แต่ยังคงใช้วิธีการ single-render เหมือนเดิม

เพื่อแสดงขั้นตอนทั้งหมด เราจะเพิ่มตัวป้อนข้อมูลเข้าไปที่คลาส `App` และปุ่ม ซึ่งถ้าปุ่มถูกคลิกจะส่งตัวอักขระไปให้ component ที่ชื่อ `Tags`

<br />

```jsx
class App extends React.Component {
  constructor(props) {
    super(props);

    this._addNewTag = this._addNewTag.bind(this);
    this.state = {
      tags: ['JavaScript', 'CSS' ],
      newTag: null
    };
  }
  _addNewTag() {
    this.setState({ newTag: this.refs.field.value });
  }
  render() {
    return (
      <div>
        <p>Add new tag:</p>
        <div>
          <input type='text' ref='field' />
          <button onClick={ this._addNewTag }>Add</button>
        </div>
        <Tags
          tags={ this.state.tags }
          newTag={ this.state.newTag } />
      </div>
    );
  }
}
```

เราใช้ state ภายใน class `App` เป็นเหมือนกับที่เก็บข้อมูลสำหรับค่าของตัวที่พึ่งถูกเพิ่มเข้าในในฟิลด์ใหม่ ทุกครั้งที่เราคลิกตัวปุ่ม ตัว React จะทำการอัปเดต state และจะไปเรียกการ re-rendering ของ component `Tags` อย่างไรก็ตามเพราะว่า `shouldComponentUpdate` React จึงไม่มีการอัปเดตใด ๆ บนหน้าจอ สิ่งอย่างเดียวที่เปลี่ยนนั่นคือเมื่อเราได้ค่าใหม่ของ prop ที่ชื่อว่า `newTag` ซึ่งอาจถูกจับมาได้ด้วย lifecycle method ตัวหนึ่งที่ชื่อว่า `componentWillReceiveProps`:

<br /><br /><br />

```jsx
class Tags extends React.Component {
  ...
  componentWillReceiveProps(newProps) {
    this.list.tagit('createTag', newProps.newTag);
  }
  ...
```

`.tagit('createTag', newProps.newTag)` คือโค้ดที่เป็น pure jQuery; `componentWillReceiveProps` คือ lifecycle method ที่เหมาะสมสำหรับการเรียกเมธอดที่มาจาก third-party library

นี่คือโค้ดที่สมบูรณ์ของ component `Tags`:

```jsx
class Tags extends React.Component {
  componentDidMount() {
    this.list = $(this.refs.list);
    this.list.tagit();
  }
  shouldComponentUpdate() {
    return false;
  }
  componentWillReceiveProps(newProps) {
    this.list.tagit('createTag', newProps.newTag);
  }
  render() {
    return (
      <ul ref='list'>
      { 
        this.props.tags.map(
          (tag, i) => <li key={ i }>{ tag } </li>
        ) 
      }
      </ul>
    );
  }
};
```

<br />

## ข้อคิด

ถึงแม้ว่า React จะเป็นคนจัดการ DOM tree เราสามารถก็ที่ยังสามารถใช้งานกับ third-party libraries และ services โดยให้ lifecycle method ต่าง ๆ เป็นตัวเชื่อมระหว่าง React กับ non-React code
