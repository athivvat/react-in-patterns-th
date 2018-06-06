# Styling React components

React เป็นไลบรารี่ที่จัดอยู่ในเลเยอร์ `View` เนื่องจากมันควบคุมการเรนเดอร์ของ markup บนเบราเซอร์ และเรารู้ว่าการจัด style ด้วย css นั้นเป็นสิ่งที่เชื่อมต่อกับ markup ที่อยู่บนเพจต่างๆอย่างแน่นหนา และมีหลายวิธีด้วยกันในการจัด style สำหรับการพัฒนาแอพพลิเคชั่นด้วย React สำหรับใน section นี้ เราจะมาดูวิธีที่นิยมมากที่สุดกัน

## The good old CSS class

การเขียนในรูปแบบ `JSX` นั้นจะมีความใกล้เคียงกับ `HTML` เนื่องจากมีชื่อ tag attribute ที่คล้ายคลึงกัน และเดิมทีเราอาจจะยังตกแต่ง style ด้วยคลาส CSS อยู่ ซึ่งมันถูกประกาศเป็นไฟล์ `.css` แยกไว้อีกไฟล์หนึ่ง จุดที่แตกต่างกันหลักๆก็คือรูปแบบ `JSX` จะใช้ tag attribute ชื่อว่า `className` ส่วนรูปแบบ `HTML` จะใช้ tag attribute ชื่อว่า `class` ยกตัวอย่างเช่น

```html
<h1 className='title'>Styling</h1>
```

## Inline styling

The inline styling works just fine. Similarly to HTML we are free to pass styles directly via a `style`  attribute. However, while in HTML the value is a string in JSX must be an object.

```js
const inlineStyles = {
  color: 'red',
  fontSize: '10px',
  marginTop: '2em',
  'border-top': 'solid 1px #000'
};

<h2 style={ inlineStyles }>Inline styling</h2>
```

Because we write the styles in JavaScript we have some limitations from a syntax point of view. If we want to keep the original CSS property names we have to put them in quotes. If not then we have to follow the camel case convention. However, writing styles in JavaScript is quite interesting and may be a lot more flexible then the plain CSS. Like for example inheriting of styles:

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

We have some basic styles in `theme` and with mix them with what is in `paragraphText`. Shortly, we are able to use the whole power of JavaScript to organize our CSS. What it matters at the end is that we generate an object that goes to the `style` attribute.

## CSS modules

[CSS modules](https://github.com/css-modules/css-modules/blob/master/docs/get-started.md) is building on top of what we said so far. If we don't like the JavaScript syntax then we may use CSS modules and we will be able to write plain CSS. Usually this library plays its role at bundling time. It is possible to hook it as part of the transpilation step but normally is distributed as a build system plugin.

Here is a quick example to get an idea how it works:

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