# การผสานกับซอฟแวร์ภายนอก (Third-party)

React อาจจะเป็นหนึ่งในตัวเลือกที่ดีทีสุดสำหรับการสร้างส่วนประสานกับผู้ใช้ (UI) การออกแบบที่ดี แล้วก็ในเรื่องของการสนับสนุนและชุมชนของนักพัฒนา อย่างไรก็ตามก็ยังมีในหลายกรณีที่เราต้องการจะใช้บริการภายนอก หรือต้องการที่จะเชื่อมต่อผสานกับอะไรสักอย่างที่มันแตกต่างไปอย่างสิ้นเชิง พวกเราทั้งหมดรู้ว่า React ทำงานอย่างหนักกับตัว DOM (Document Object Model) จริง ๆ ของเว็บ ซึ่งพวก HTML, XML มักจะใช้การเก็บ Document เป็นอย่างที่กล่าวมา และการควบคุมอะไรก็ตามที่จะแสดงผลออกมาทางหน้าจอเป็นพื้นฐาน นั่นก็คือเหตุผลที่การเชื่อมต่อผสานของส่วนประกอบจากซอฟแวร์ภายนอก (third-party) ค่อนข้างที่จะต้องใช้เทคนิคที่อาจจะยุ่งยาก ในส่วนนี้เราจะแสดงวิธีที่จะรวม React และส่วนเสริมการประสานงานกับผู้ใช้ (jQuery's UI plugin) และค่อย ๆ ทำมันอย่างปลอดภัย

## ตัวอย่าง

กระผมได้เลือกใช้ [*tag-it*](https://github.com/aehlke/tag-it) ซิ่งเป็นส่วนเสริมตัวนึงของ jQuery มาใช้เป็นตัวอย่างนะครับ มันเอาไว้แปลงแท็ก ul ที่เอาไว้แสดงผลข้อมูลรายการที่ไม่เป็นลำดับให้กลายเป็นตัวป้อนข้อมูลที่จะไว้ใช้ในการจัดการแท็ก

จากภาษา HTML ด้านล่างนี้:

```html
<ul>
  <li>JavaScript</li>
  <li>CSS</li>
</ul>
```

ไปแสดงผลเป็น:

![tag-it](./tag-it.png)

เพื่อให้มันทำงานได้ เราจำเป็นจะต้องมี jQuery, jQuery UI และที่ขาดไม่ได้ *tag-it*; tag-it มันใช้งานประมาณนี้ครับ:

```jsx
$('<dom element selector>').tagit();
```

อธิบายก็คือเราเลือก DOM element และไปเรียกใช้งานฟังก์ชันที่ชื่อ `target()`

เอาละครับ มาสร้าง React app ง่าย ๆ ขึ้นมาตัวนึง ที่จะมาลองใช้กับตัวส่วนขยาย:

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

เข้าไปที่คลาสที่ชื่อว่า `App` มันใช้งานตัวคอมโพเนนท์ที่ชื่อว่า `Tags` ที่จะทำการแสดงผลเจ้าตัวรายการที่ไม่เป็นลำดับ (unordered list) ด้วยการส่งค่าผ่าน prop ที่ชื่อว่า `tags` แล้วเมื่อ React ทำการแสดงรายการที่ว่าบนหน้าจอ เราจะรู้ว่าเรามีแท็ก `<ul>` เพื่อเราสามารถgเชื่อมมันเข้ากับ jQuery plugin

## บังคับ single-render

สิ่งแรกที่เราจะต้องคือการบังคับ single-render ของคอมโพเนนท์ `Tags` นั่นเพราะเมื่อ React เพิ่ม element ต่าง ๆ เข้าไปที่ DOM จริง ๆ (actual DOM) เราต้องการที่จะส่งการควบคุม element ต่าง ๆ ไปให้ jQuery ถ้าเราข้ามทั้งสอง React และ jQuery จะทำงานอยู่บน DOM ตัวเดียวกัน โดยไม่รู้ซึ่งกันและกัน เพื่อให้ได้ single-render เราจะต้องใช้เมธอดที่อยู่ใน lifecycle ของ React ที่ชื่อว่า `shouldComponentUpdate` อย่างเช่นโค้ดด้านล่างนี้: 

```jsx
class Tags extends React.Component {
  shouldComponentUpdate() {
    return false;
  }
  ...
```

โดยการส่งค่ากลับมาเป็น `false` เสมออย่างนี้ เรากล่าวว่าคอมโพเนนท์ของเราจะไม่แสดงผลใหม่อีก ถ้ากำหนด `shouldComponentUpdate` ที่ถูกนำมาใช้โดย React เพื่อที่จะเข้าใจว่ามันเป็นตัวทำให้เกิดการ `render` หรือไม่ นั่นคือสิ่งที่ดีเลิศสำหรับกรณีของเรา เพราะว่าเราต้องการที่จะวางโครงสร้างบนหน้าที่ใช้ React แต่เราไม่ต้องการที่จะวางใจกับมันหลังจากนั้น

## การเตรียมพร้อมสำหรับส่วนขยาย

React ได้ให้ [API](https://facebook.github.io/react/docs/refs-and-the-dom.html) มาตัวนึงสำหรับการเข้าถึง actual DOM nodes เราจะต้องใช้ attribute ที่ชื่อว่า `ref` กับตัว node และถัดมาก็เป็นการเข้าถึงตัว node ด้วย `this.refs` ซึ่ง `componentDidMount` เป็น lifecycle method ที่เหมาะสำหรับการเตรียมการให้ส่วนขยาย *tag-it* นั่นเป็นเพราะว่ามันจะถูกเรียกเมื่อ React สร้างผลลัพธ์ของเมธอด `render`

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

ตัวโค้ดที่อยู่ด้านบนกับเมธอด `shouldComponentUpdate` นำไปสู่การที่ React เรนเดอร์ตัว `<ul>` กับสองไอเท็ม และจากนั้น *tag-it* จะทำการแปลงมันเพื่อทำการแก้ไข tag widget

## การควบคุมส่วนขยายด้วย React

สมมติว่าเราต้องการที่จะโปรแกรมเพิ่ม แท็กตัวใหม่ที่กำลังทำงานอยู่แล้วกับ *tag-it* การกระทำดังกล่าวจะถูกเรียกโดย React คอมโพเนนท์ และต้องใช้ jQuery API; เราจะต้องหาทางที่จะสื่อสารข้อมูลกับ `Tags` คอมโพเนนท์ แต่ยังคงเก็บวิธีการเข้าไป single-render

เพื่อแสดงขั้นตอนทั้งหมด เราจะเพิ่มตัวป้อนข้อมูลเข้าไปที่คลาส `class` และปุ่ม ซึ่งถ้าปุ่มถูกคลิกจะส่งตัวอักขระไปให้คอมโพเนนท์ที่ชื่อ `Tags`

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

เราใช้ state ภายในเป็นเหมือนกับที่เก็บข้อมูลสำหรับค่าของตัวที่พึ่งถูกเพิ่มเข้าในในฟิลด์ใหม่ ทุกครั้งที่เราคลิกตัวปุ่ม ตัว React จะทำการอัปเดต state และจะไปเรียกการ re-rendering ของคอมโพเนนท์ `Tags` อย่างไรก็ตามเพราะว่า `shouldComponentUpdate` เราจึงไม่มีการอัปเดตใด ๆ บนหน้าจอ สิ่งอย่างเดียวที่เปลี่ยนนั่นคือเราได้ค่าใหม่ของ prop ที่ชื่อว่า `newTag` ซึ่งอาจถูกจับมาได้ด้วย lifecycle method ตัวหนึ่งที่ชื่อว่า `componentWillReceiveProps`:

<br /><br /><br />

```jsx
class Tags extends React.Component {
  ...
  componentWillReceiveProps(newProps) {
    this.list.tagit('createTag', newProps.newTag);
  }
  ...
```

`.tagit('createTag', newProps.newTag)` คือโค้ดที่เป็น pure jQuery; `componentWillReceiveProps` คือที่ที่ดีสำหรับการเรียกเมธอดที่มาจาก third-party library

นี่คือโค้ดที่สมบูรณ์ของ `Tags` คอมโพเนนท์:

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

## สรุปสุดท้าย

ถึงแม้นว่า React เป็นการจัดการเจ้า DOM tree เราสามารถที่จะเชื่อมผสานกับ third-party libraries และ services; lifecycle method ที่ให้เรามาเพียงพอที่จะควบคุมกระบวนการการแสดงผล ซึ่งสิ่ง lifecycle method จะเป็นสะพานสมบูรณ์ที่เชื่อมระหว่าง React และโค้ดที่ไม่ใช่ React
