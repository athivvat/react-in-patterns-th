# การตกแต่ง React components

React นั้นเป็นส่วนแสดงผล ซึ่งเป็นการควมคุม Markup ที่จะแสดงผลในเบราว์เซอร์ เรามักจะใช้ CSS ในการตกแต่งหน้า Markup ของเรา มีหลายวิธีมากในการจัดการกับ Styling บนแอพพลิเคชั่น React และในบทนี้ เราจะมาพูดถึงวิธีการที่นิยมกัน

## CSS Class ที่ดีในทุกยุคสมัย

JSX Syntax นั้นมีความใกล้เคียงกับภาษา HTML ซึ่งนั่นก็คือเรายังคงใช้ Attribute ต่าง ๆ เหมือนกัน และเราอาจจะยังคงใช้ CSS Class ในการ Styling โดยที่ Class ต่างๆ ถูกประกาศจากไฟล์ `.css` โดยมีข้อแตกต่างอย่างนึงก็คือต้องใช้ `className` ไม่ใช่ `class` เช่น

```
<h1 className='title'>Styling</h1>
``` 

## Inline styling

การทำ Inline CSS ก็สามารถทำได้ดีเช่นเดียวกับ HTML ที่เราสามารถส่งค่า Parameter ต่าง ๆ ได้โดยตรงผ่าน Attribute `style` แต่อย่างไรก็ตาม ใน JSX นั้นเราจะต้องกำหนด Styling ด้วย Object แตกต่างจาก HTML ที่กำหนดเป็น String

```js
const inlineStyles = {
  color: 'red',
  fontSize: '10px',
  marginTop: '2em',
  'border-top': 'solid 1px #000'
};

<h2 style={ inlineStyles }>Inline styling</h2>
```

เพราะว่าเราเขียน Style ใน Syntax ของ JavaScript เราจึงมีข้อจำกัดของ Syntax
หากเราต้องการเขียน CSS Property ในแบบของ CSS ดั้งเดิมนั้น  เราจะต้องเขียนภายใน Quote ( เครื่องหมาย ", ' ) ถ้าไม่เช่นนั้นคุณก็จะต้องเขียนตามหลัก Camel Case อย่างไรก็ตาม Styling ใน JavaScript นั้นมีความน่าสนใจและยืดหยุ่นได้หลากหลายวิธีกว่า CSS ปกติทั่วไป (เช่น Plain CSS ใน HTML) ดังในตัวอย่างด้านล่างนี้ เราส่งผ่าน Property จาก Style หนึ่งไปยังอีก Style หนึ่ง:

```js
const theme = {
  fontFamily: 'Georgia',
  color: 'blue'
};
const paragraphText = {
  // ES2018 object spread
  ...theme,
  fontSize: '20px'
};
```

เรามี Style ชุดนึงใน `theme` และเราก็เรียกใช้มันภายใน Style ของ `paragraphText` อธิบายง่าย ๆ ก็คือ เราสามารถใช้ความสามารถของ JavaScript ในการจัดการ CSS ของเรา สิ่งที่เราต้องการให้คุณเห็นคือสุดท้ายเราได้สร้าง Object หนึ่ง ซึ่งมันจะไปแทรกตัวอยู่ใน Attribute `style`

## CSS modules

[CSS modules](https://github.com/css-modules/css-modules/blob/master/docs/get-started.md) นั้นสร้างขึ้นจากแนวคิดของสิ่งที่เราได้กล่าวไปก่อนหน้านี้ ถ้าเราไม่ชอบการเขียน CSS ภายใต้ Syntax ของ JavaScript เราสามารถใช้ CSS Module ที่ทำให้เราสามารถเขียน CSS ในรูปแบบและ Syntax ของ CSS ได้ ( ดังที่กล่าวไปว่าการเขียน CSS โดยที่ไม่มี CSS module นั้นจะต้องเขียนใน Syntax ของ JavaScript )
ปกติแล้ว Library นี้จะจัดการงานของมันในช่วง Building Time มันเป็นไปได้ที่เราจะเข้าใจว่ามันคือส่วนหนึ่งของกระบวนการ [Transpilation](https://scotch.io/tutorials/javaScript-transpilers-what-they-are-why-we-need-them) แต่โดยปกติแล้วมันก็คือ build system plug-in ชนิดหนึ่ง

นี่คือตัวอย่างเล็ก ๆ ที่จะช่วยให้คุณเข้าใจว่ามันทำงานอย่างไร:

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

ตามปกติแล้ววิธีนี้จะไม่สามารถทำได้ แต่ด้วยพลังแห่ง CSS Module เราสามารถ import ไฟล์ CSS และเรียกใช้ Class ที่ประกาศไว้ในไฟล์ CSS Module ได้

และดังที่เราได้กล่าวไปว่า Plain CSS ใน React นั้น ไม่ได้เหมือนกับ CSS ธรรมดาบน HTML มันทำให้คุณสามารถใช้งานเทคนิคบางอย่างที่เป็นประโยชน์กับคุณได้ ดังตัวอย่าง:

```
.title {
  composes: mainColor from "./brand-colors.css";
}
```

## Styled-components

[styled-components](https://www.styled-components.com/) นั้นต่างออกไป แทนที่จะเป็น Inline Style ที่เกิดจาก Library เราใช้มันเพื่อทำให้โค้ดของเราดูดีมากขึ้น เช่น เราอาจจะสร้าง Component `Link` ซึ่งมี Style และมีการใช้การใช้งานเหมือนกับ `<a>`

```js
const Link = styled.a`
  text-decoration: none;
  padding: 4px;
  border: solid 1px #999;
  color: black;
`;

<Link href='http://google.com'>Google</Link>
```

เช่นเดิมที่เรามีวิธีในการขยายเพิ่ม Class อีกด้วย เราอาจจะยังคงใช้ Component `Link` แต่เปลี่ยนสีตัวอักษรได้ดังนี้:

```js
const AnotherLink = styled(Link)`
  color: blue;
`;

<AnotherLink href='http://facebook.com'>Facebook</AnotherLink>
```

ตามความคิดเห็นผมแล้ว styled-components เป็นวิธีการที่น่าสนใจที่สุดแล้วในการ Styling ใน React มันง่ายมากในการสร้าง Components สำหรับทุก ๆ อย่างและลดความยุ่งยากในการ Styling ลง ถ้าบริษัทของคุณมีความพร้อมในการทำระบบดีไซน์ ( Design System ) และการพัฒนาระบบด้วย styled-components นี่คงจะเป็นทางเลือกที่ดีที่สุดแล้ว

## ข้อคิดสุดท้าย

มีหลายทางเลือกมาก ๆ ในการตกแต่งโปรเจคของคุณ ผมได้ทดลองมามากมายและผมคงจะบอกได้ว่าไม่มีวิธีไหนถูกหรือผิด มีทางเลือกให้คุณได้ใช้มากมาย คุณควรจะเลือกสักวิธีหนึ่งที่ดีที่สุดในโปรเจคของคุณ