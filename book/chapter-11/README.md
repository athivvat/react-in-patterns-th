# การตกแต่ง React components

React นั้นเป็นส่วนแสดงผล ซึ่งเป็นการควบคุมรูปแบบนึงของ Markup ซึ่งทำงานบนเบราว์เซอร์ และเราก็รู้แล้วว่าการใช้ CSS บนเว็บของเรานั้นถูกเชื่อมต่อกันโดยสมบูรณ์กับ มีหลายวิธีมากในการจัดการกับ Styling บนแอพพลิเคชั่น React และในบทนี้ เราจะมาพูดถึงวิธีการที่นิยมกัน

## CSS Class ที่ดีในทุกยุคสมัย

JSX Syntax นั้นมีความใกล้เคียงกับภาษา HTML ซึ่งนั่นก็คือเรายังคงใช้ Attribute ที่เหมือนกัน และเราอาจจะใช้ CSS Class ในการตกแต่ง โดยที่ Class ต่างๆ ถูกประกาศจากไฟล์ `.css` โดยมีข้อแม้อย่างเดียวก็คือต้องใช้ `className` ไม่ใช่ `class` เช่น

```
<h1 className='title'>4 ปีแล้วนะ</h1>
``` 

## Inline styling

การทำ Inline css ก็สามารถทำได้เช่นกัน เหมือนกับ HTML ที่เราสามารถส่งค่าต่างๆได้โดยตรงผ่าน Attribute `style` แต่อย่างไรก็ตาม ค่า value ที่เป็นค่า string นั้น ใน JSX จะต้องเป็น object

```js
const inlineStyles = {
  color: 'red',
  fontSize: '10px',
  marginTop: '2em',
  'border-top': 'solid 1px #000'
};

<h2 style={ inlineStyles }>Inline styling</h2>
```

เพราะว่าเราเขียน style ใน Javascript เราจึงมีข้อจำกัดจาก Syntax หากเราต้องการเขียน CSS Property ใบแบบของ CSS เราจะต้องเขียนภายใน Quote ถ้าไม่เช่นนั้นคุณก็จะต้องเขียนตามหลัก Camel case อย่างไรก็ตาม การเขียน Style ใน Javascript นั้นมีความน่าสนใจและยืดหยุ่นได้หลากหลายวิธีกว่า CSS ปกติ ดังในตัวอย่างนี้ เราส่งผ่าน Property จาก Style หนึ่งไปยังอีก Style หนึ่ง:

```js
const theme = {
  fontFamily: 'Georgia',
  color: 'blue'
};
const paragraphText = {
  ...theme,
  fontSize: '20px'
};
```

เรามี Basic Style ใน `theme` และเราก็รวมมันกับสิ่งที่อยู่ใน `paragraphText` อธิบายง่ายๆก็คือ เราสามารถใช้ความสามารถของ Javascript ในการจัดการ CSS ของเรา สิ่งที่สำคัญคือช่วงสุดท้ายเราได้สร้าง object ซึ่งมันจะไปแทรกตัวอยู่ใน Attribute `style`

## CSS modules

[CSS modules](https://github.com/css-modules/css-modules/blob/master/docs/get-started.md) นั้นสร้างขึ้นจากแนวคิดของสิ่งที่เราได้กล่าวไปก่อนหน้านี้ ถ้าเราไม่ชอบ syntax ของ Javascript เราสามารถเลือก CSS Module ที่ทำให้เราสามารถเขียน CSS แบบธรรมดาได้ ปกติแล้ว Library นี้จะจัดการงานของมันในช่วง Building Time
เราอาจจะคิดว่ามันเป็นส่วนหนึ่งของการทำ Transpilation แต่ปกติแล้วมันคือส่วนหนึ่งของ plug-in ของการสร้างระบบ

นี่คือตัวอย่างเล็กๆ ที่จะช่วยให้คุณเข้าใจว่ามันทำงานอย่างไร:

<br /><br />

```js
/* style.css */
.title {
  color: green;
}

// App.jsx
import styles from "./style.css";

function App() {
  return <h1 style={ styles.title }>Hello world</h1>;
}
```

That is not possible by default but with CSS modules we may import directly a plain CSS file and use the classes inside.

And when we say *plain CSS* we don't mean that it is exactly like the normal CSS. It supports some really helpful composition techniques. For example:

```
.title {
  composes: mainColor from "./brand-colors.css";
}
```

## Styled-components

[Styled-components](https://www.styled-components.com/) took another direction. Instead of inlining styles the library provides a React component. We then use this component to represent a specific look and feel. For example, we may create a `Link` component that has certain styling and use that instead of the `<a>` tag.

```js
const Link = styled.a`
  text-decoration: none;
  padding: 4px;
  border: solid 1px #999;
  color: black;
`;

<Link href='http://google.com'>Google</Link>
```

There is again a mechanism for extending classes. We may still use the `Link` component but change the text color like so:

```js
const AnotherLink = styled(Link)`
  color: blue;
`;

<AnotherLink href='http://facebook.com'>Facebook</AnotherLink>
```

By far for me styled-components are probably the most interesting approach for styling in React. It is quite easy to create components for everything and forget about the styling. If your company has the capacity to create a design system and building a product with it then this option is probably the most suitable one.

## Final thoughts

There are multiple ways to style your React application. I did experienced all of them in production and I would say that there is no right or wrong. As most of the stuff in JavaScript today you have to pick the one that fits better in your context.