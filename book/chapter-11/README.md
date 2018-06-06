# การตกแต่ง React components

React นั้นเป็นส่วนแสดงผล ซึ่งเป็นการควบคุมรูปแบบนึงของ Markup ซึ่งทำงานบนเบราว์เซอร์ และเราก็รู้แล้วว่าการใช้ CSS บนเว็บของเรานั้นถูกเชื่อมต่อกันโดยสมบูรณ์กับ มีหลายวิธีมากในการจัดการกับ Styling บนแอพพลิเคชั่น React และในบทนี้ เราจะมาพูดถึงวิธีการที่นิยมกัน

## CSS Class ที่ดีในทุกยุคสมัย

JSX Syntax นั้นมีความใกล้เคียงกับภาษา HTML ซึ่งนั่นก็คือเรายังคงใช้ Attribute ที่เหมือนกัน และเราอาจจะใช้ CSS Class ในการทำ Styling โดยที่ Class ต่างๆ ถูกประกาศจากไฟล์ `.css` โดยมีข้อแม้อย่างเดียวก็คือต้องใช้ `className` ไม่ใช่ `class` เช่น

```
<h1 className='title'>4 ปีแล้วนะ</h1>
``` 

## Inline styling

การทำ Inline CSS ก็สามารถทำได้เช่นกัน เหมือนกับ HTML ที่เราสามารถส่งค่า Parameter ต่างๆได้โดยตรงผ่าน Attribute `style` แต่อย่างไรก็ตาม ค่า value ที่เป็นค่า string นั้น ใน JSX จะต้องเป็น object

```js
const inlineStyles = {
  color: 'red',
  fontSize: '10px',
  marginTop: '2em',
  'border-top': 'solid 1px #000'
};

<h2 style={ inlineStyles }>Inline นี้สีแดงห่างบน ๆ</h2>
```

เพราะว่าเราเขียน Style ใน Syntax ของ Javascript เราจึงมีข้อจำกัดจาก Syntax หากเราต้องการเขียน CSS Property ใบแบบของ CSS เราจะต้องเขียนภายใน Quote ( เครื่องหมาย ", ' ) ถ้าไม่เช่นนั้นคุณก็จะต้องเขียนตามหลัก Camel Case อย่างไรก็ตาม Styling ใน Javascript นั้นมีความน่าสนใจและยืดหยุ่นได้หลากหลายวิธีกว่า CSS ปกติทั่วไป (เช่น Plain CSS ใน HTML) ดังในตัวอย่างด้านล่างนี้ เราส่งผ่าน Property จาก Style หนึ่งไปยังอีก Style หนึ่ง:

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

เรามี Style ใน `theme` และเราก็รวมมันกับสิ่งที่อยู่ใน `paragraphText` อธิบายง่ายๆก็คือ เราสามารถใช้ความสามารถของ Javascript ในการจัดการ CSS ของเรา สิ่งที่สำคัญคือช่วงสุดท้ายเราได้สร้าง Object ซึ่งมันจะไปแทรกตัวอยู่ใน Attribute `style`

## CSS modules

[CSS modules](https://github.com/css-modules/css-modules/blob/master/docs/get-started.md) นั้นสร้างขึ้นจากแนวคิดของสิ่งที่เราได้กล่าวไปก่อนหน้านี้ ถ้าเราไม่ชอบ Syntax ของ Javascript เราสามารถเลือก CSS Module ที่ทำให้เราสามารถเขียน CSS แบบธรรมดาได้ ปกติแล้ว Library นี้จะจัดการงานของมันในช่วง Building Time
เราอาจจะคิดว่ามันเป็นส่วนหนึ่งของการทำ Transpilation แต่ปกติแล้วมันคือส่วนหนึ่งของ Plug-in ของการสร้างระบบ

นี่คือตัวอย่างเล็ก ๆ ที่จะช่วยให้คุณเข้าใจว่ามันทำงานอย่างไร:

```js
/* style.css */
.title {
  // ณ จุด ๆ นี้ คุณไม่ต้องครอบ green ด้วย Quote อีกต่อไป
  color: green;
}

// App.jsx
import styles from "./style.css";

function App() {
  return <h1 style={ styles.title }>Hello world</h1>;
}
```

ตามปกติแล้ววิธีนี้จะไม่สามารถทำได้ แต่ด้วยพลังแห่ง CSS Module เราสามารถ import ไฟล์ CSS และเรียกใช้ Class ได้

และดังที่เราได้กล่าวไปว่า Plain CSS ใน React นั้น ไม่ได้เหมือนกับ CSS ธรรมดาบน HTML มันสามารถใช้งานเทคนิคบางอย่างที่เป็นประโยชน์กับคุณได้ ดังตัวอย่าง:

```
.title {
  composes: mainColor from "./brand-colors.css";
}
```

## Styled-components

[Styled-components](https://www.styled-components.com/) นั้นต่างออกไป แทนที่จะเป็น Inline Style ที่เกิดจาก Library เราใช้มันเพื่อทำให้โค้ดของเราดูดีมากขึ้น เช่น เราอาจจะสร้างคอมโพเนนท์ `Link` ซึ่งมี Style และมีการใช้การใช้งานเหมือนกับ `<a>`

```js
const Link = styled.a`
  text-decoration: none;
  padding: 4px;
  border: solid 1px #999;
  color: black;
`;

<Link href='https://app.cupt.net/tcas/round3.php'>TCAS</Link>
```

นี่คือรูปแบบทั่วไปที่เอาไว้ใช้สร้าง Class เรายังคงสามารถใช้ Component `Link` แต่เปลี่ยนสีตัวอักษรได้ดังนี้:

```js
const AnotherLink = styled(Link)`
  color: blue;
`;

<AnotherLink href='http://www.mdes.go.th/'>Try to DoS me!</AnotherLink>
```

ตามความคิดเห็นผมแล้ว styled-components เป็นวิธีการที่น่าสนใจที่สุดแล้วในการตกแต่งใน React มันง่ายมากในการสร้างทุก ๆ อย่างและลดความยุ่งยากในการ Styling ลง ถ้าบริษัทของคุณมีความเข้าใจและพร้อมในการทำระบบดีไซน์และและการทำระบบด้วย Styled-components นี่คงจะเป็นทางเลือกที่ดีที่สุดแล้ว

## ความเห็นสุดท้าย

มีหลายทางเลือกมาก ๆ ในการตกแต่งโปรเจคของคุณ ผมได้ทดทดลองมามากมายและผมคงจะบอกได้ว่าไม่มีวิธีไหนถูกหรือผิด มีทางเลือกให้คุณได้ใช้มากมาย คุณควรจะเลือกสักวิธีหนึ่งที่ดีที่สุดในโปรเจคของคุณ