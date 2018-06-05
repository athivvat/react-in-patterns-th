# Controlled Input และ Uncontrolled Input

*Controlled Input* และ *Uncontrolled Input* จะถูกนำไปใช้ในการจัดการข้อมูล หรือ action ต่างๆของ form *Controlled Input* นั้นค่าของ input จะถูกกำหนดด้วยข้อมูลจากภาคนอก ที่เรามักจะใช้ค่านี้จากแหล่งข้อมูลที่หนึ่งที่มีค่าความจริงเพียงหนึ่งเดียวเท่านั้น (Single source of thruth) ดังตัวอย่างด้านล่าง component `App` มี element `<input>` อยู่หนึ่งตัว ซึ่งเป็น *Controlled Input* 

field which is *controlled*:
These two terms *controlled* and *uncontrolled* are very often used in the context of forms management. *controlled* input is an input that gets its value from a single source of truth. For example the `App` component below has a single `<input>` field which is *controlled*:
