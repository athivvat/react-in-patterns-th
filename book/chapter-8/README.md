# Flux

ผมมักจะเขียนโค้ดทำให้มันดูง่าย ผมไม่ได้หมายความว่าเขียนโค้ดให้น้อย เพราะการเขียนโค้ดให้น้อยมันก็ไม่ได้หมายความว่ามันจะทำงานง่าย ผมเชื่อว่าปัญหาที่ใหญ่ในวงการของการพัฒนาซอฟต์แวร์นั้นมาจากความซับซ้อนที่ไม่จำเป็น ความซับซ้อนนี้เป็นผลมาจากงานของของเราเองซึ่งมันเป็นสิ่งที่เป็นนามธรรม เหมือนกับการที่เราอะไรใส่อะไรบางอย่างในกล่องดำ (black box) และเราก็คาดหวังว่ามันจะทำงานร่วมกัน

[Flux](http://facebook.github.io/flux/) เป็นรูปแบบหนึ่งของการออกแบบสถาปัตยกรรมสำหรับการสร้างส่วนติดต่อผู้ใช้ ถูกเผยแพร่โดย Facebook ในงานสัมนา [F8](https://youtu.be/nYkdrAPrdcw?t=568) หลังจากนั้นหลายบริษัทได้นำไปใช้และดูเหมือนว่ามันวิธีการที่ดีในการพัฒนา Front-end apps รูปแบบ Flux ถูกนำมาใช้ควบคู่กับ React บ่อยมาก ตัวผมเองได้ใช้ React+Flux/Redux ในงานประจำของผม และผมบอกได้เลยว่ามันง่ายและยืดหยุ่นจริงๆ รูปแบบดังกล่าวช่วยให้สร้างแอปได้เร็วขึ้นและในเวลาเดียวกัน ก็ช่วยให้โค้ดดูเป็นระเบียบเรียบร้อยมากขึ้น

## สถาปัตยกรรม Flux และลักษณะสำคัญ

![Basic flux architecture](./fluxiny_basic_flux_architecture.jpg)

พระเอกของ Flux คือ *dispatcher* คอยทำหน้าที่เป็นจุดเชื่อมกันสำหรับ event ทั้งหมดของระบบ หน้าที่ของมันคือรอรับการแจ้งเตือนเมื่อเราได้เรียก *action* และส่งไปยัง *store* เพื่อทำการตรสจสอบว่าจะต้องทำการเปลี่ยนแปลงของ state หรือไม่ เมื่อมีการเปลี่ยนแปลงเกิดขึ้นก็ทำการ render *view* (React components) ใหม่ ถ้าเราเปรียบเทียบ Flux กับ MVC อาจจะพูดได้ว่า store เปรียบเสมือนกับ model ที่คอยเก็บข้อมูลและวิธีการในการเปลี่ยนแปลงข้อมูล

Action ที่มายัง dispatcher นั้นมาจากทั้งส่วนของ view และส่วนอื่น ๆ ของระบบ เปรียบเสมือนกับเป็นบริการ (service) ยกตัวอย่างเช่นโมดูลที่ทำการร้องขอ HTTP เมื่อได้รับการตอบกลับมาจะทำการดำเนินการบางอย่าง เพื่อทำการบอกว่าการร้องขอนั้นได้สำเร็จแล้ว


## การใช้งานสถาปัตยกรรม Flux

เช่นเดียวกับแนวคิดยอดนิยมอื่น ๆ Flux ก็มีรูปแบบการนำไปใช้ที่[หลากหลาย](https://medium.com/social-tables-tech/we-compared-13-top-flux-implementations-you-won-t-believe-who-came-out-on-top-1063db32fe73) โดยทั่วไปวิธีที่ดีที่สุดในการทำความเข้าใจกับเทคโนโลยีคือการนำไปปฏิบัติ ในส่วนต่อไปนี้เราจะสร้างไลบรารีที่มีฟังก์ชัน helper เพื่อสร้างรูปแบบ Flux

### Dispatcher

ในหลายกรณีเราจำเป็นต้องมี dispatcher อันเดียว เพราะมันทำหน้าที่เป็นตัวเชื่อมกันกับตัวอื่น ๆ ที่เหลือให้เหมือนกับว่าเพียงแค่อันเดียว dispatcher จะรู้จักกับสองสิ่งนี้คือ action และ store ตัว actions นั้นถูกส่งแค่ไปหา stores เฉยๆ ทำให้เราไม่จำเป็นต้องเก็บมันไว้ ส่วน store สามารถถูก track ได้ภายใน dispatcher เพื่อสามารถดำเนินการกับข้อมูลได้:

![the dispatcher](./fluxiny_the_dispatcher.jpg)

ผมจะเริ่มต้นด้วย:

```js
var Dispatcher = function () {
  return {
    _stores: [],
    register: function (store) {  
      this._stores.push({ store: store });
    },
    dispatch: function (action) {
      if (this._stores.length > 0) {
        this._stores.forEach(function (entry) {
          entry.store.update(action);
        });
      }
    }
  }
};
```

สิ่งแรกที่ควรทราบก็คือเราคาดว่า method update จะมีอยู่ใน store ที่ได้รับมา หากไม่มีควรจะทำการส่ง error กลับไป:

```js
register: function (store) {
  if (!store || !store.update) {
    throw new Error('You should provide a store that has an `update` method.');
  } else {
    this._stores.push({ store: store });
  }
}
```

<br />

### ผูก view กับ store

ขั้นตอนต่อไปคือการเชื่อมต่อส่วนผู้ใช้กับ store เพื่อให้เราสามารถแสดงผลใหม่เมื่อสถานะของ store มีการเปลี่ยนแปลง

![Bounding the views and the stores](./fluxiny_store_change_view.jpg)


#### การใช้งาน helper

การใช้งาน Flux ในบางรูปแบบจะมีฟังก์ชัน helper ในการทำงาน ตัวอย่างเช่น

```js
Framework.attachToStore(view, store);
```
อย่างไรก็ตามผมไม่ชอบวิธีนี้มากนัก เพื่อให้ `attachToStore` ทำงานได้อย่างถูกต้องเราจำเป็นต้องใช้ API แบบพิเศษใน view และ store ดังนั้นเราจำเป็นต้องกำหนด public method ใหม่อย่างเข้มงวด หรือพูดอีกอย่างหนึ่งคือ view และ store ของคุณควรมี API ดังกล่าวเพื่อให้สามารถเชื่อมต่อกันได้
ถ้าเราใช้วิธีนี้เราอาจจะกำหนดคลาสหลักที่เป็นส่วยขยายเพื่อที่จะไม่สร้างความสับสนกับรายละเอียดของ Flux ให้กับผู้พัฒนา กลายเป็นว่าทุกคลาสควรจะ extend มาจากคลาสของเรา
มันไม่ใช่ความคิดที่ดีเพราะผู้พัฒนาอาจตัดสินใจเปลี่ยนไปใช้ Flux รูปแบบอื่น และต้องการแก้ไขทุกอย่าง

<br /><br />

#### การใช้งาน mixin

เกิดอะไรขึ้นถ้าเราใช้ [mixins](https://facebook.github.io/react/docs/reusable-components.html#mixins)

```js
var View = React.createClass({
  mixins: [Framework.attachToStore(store)]
  ...
});
```
นี่เป็นวิธีที่ดีในการกำหนดลักษณะการทำงานสำหรับ component ดังนั้นในทางทฤษฎีเราอาจสร้าง mixin มาผูกกับ component ของเรา ถ้าพูดตามตรงผมไม่คิดว่านี่เป็นความคิดที่ดี และ[ดูเหมือนว่าไม่ใช่แค่ผมที่คิดแบบนี้](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750) เหตุผลที่ผมไม่ชอบ mixin คือมันปรับเปลี่ยน components โดยวิธีที่ไม่สามารถคาดเดาได้ ผมไม่รู้ว่าเกิดอะไรขึ้นเบื้องหลังการทำงานของมัน ดังนั้นผมจึงข้ามวิธีการนี้

#### การใช้งาน context

อีกเทคนิคหนึ่งที่อาจตอบโจทย์ได้คือ [context](https://facebook.github.io/react/docs/context.html) เป็นวิธีที่จะ pass props ไปยัง component ย่อยโดยไม่จำเป็นต้องส่งลงไปทีละขั้น Facebook แนะนำให้ใช้ context ในกรณีที่เราส่งข้อมูลไปยัง component ย่อยที่ถูกซ้อนกันหลายชั้น

> บางครั้ง คุณต้องการส่งข้อมูลไปยัง component โดยไม่ต้อง pass props ในแต่ละระดับด้วยตนเอง context จะช่วยให้คุณทำแบบนั้นได้ง่ายขึ้น

ผมเห็นความคล้ายคลึงกันกับ mixins ตัว context ถูกกำหนดไว้ที่ไหนสักที่ด้านบน component และคอยส่งข้อมูลไปยัง component ย่อยทุกตัวอย่างน่าอัศจรรย์ โดยที่ไม่รู้ว่าข้อมูลมาจากที่ไหน

<br /><br /><br />

#### แนวคิด Higher-Order components

รูปแบบ Higher-Order components ถูก[เสนอ](https://gist.github.com/sebmarkbage/ef0bf1f338a7182b6775)โดย Sebastian Markb&#229;ge จะจะเกี่ยวกับการสร้าง wrapper component ที่จะรีเทิร์น component ออกมา โดยที่สามารถเพิ่มคุณสมบัติให้มันการเดินการบางอย่างเข้าไปด้วย ตัวอย่างเช่น:

```js
function attachToStore(Component, store, consumer) {
  const Wrapper = React.createClass({
    getInitialState() {
      return consumer(this.props, store);
    },
    componentDidMount() {
      store.onChangeEvent(this._handleStoreChange);
    },
    componentWillUnmount() {
      store.offChangeEvent(this._handleStoreChange);
    },
    _handleStoreChange() {
      if (this.isMounted()) {
        this.setState(consumer(this.props, store));
      }
    },
    render() {
      return <Component {...this.props} {...this.state} />;
    }
  });
  return Wrapper;
};
```

`Component` คือ view ที่เราต้องการจะแนบกับ `store` ฟังก์ชั่น `consumer` จะระบุว่าควรเก็บอะไรใน store และส่งไปที่ view การใช้ฟังก์ชันข้างต้นอย่างง่ายเป็นดังนี้:

```js
class MyView extends React.Component {
  ...
}

ProfilePage = connectToStores(MyView, store, (props, store) => ({
  data: store.get('key')
}));

```

นี่เป็นรูปแบบที่น่าสนใจ view จะดึงข้อมูลจาก store ไม่ใช่ให้ store กำหนดว่าจะส่งอะไรออกไป และแน่นอนว่ามันมีทั้งข้อดีและข้อเสีย
ข้อดีคือ ทำให้ง่ายต่อการจัดเก็บ ซึ่ง store จะทำหน้่าที่เพียงแค่เปลี่ยนแปลงข้อมูลและคอยบอกว่าข้อมูลได้ถูกแก้ไขแล้ว มันไม่ได้มีหน้าที่รับผิดชอบต่อการส่งข้อมูลอีกต่อไป ข้อเสียของวิธีการนี้คือ อาจเป็นไปได้ว่าเราจะมี component (wrapper) หลาย ๆ อันที่เกี่ยวข้อง นอกจากนี้เรายังต้องมีสามสิ่งนี้คือ view, store และ consumer วางไว้ในที่เดียวกัน เพื่อให้เแต่ละส่วนสามารถเชื่อมต่อกันได้

#### ตัวเลือกของผม

ผมเลือกวิธีสุดท้าย higher-order components มีความใกล้เคียงกับสิ่งที่ผมต้องการแล้ว ให้ view กำหนดสิ่งที่ต้องการ ความเข้าใจที่มีอยู่เกี่ยวกับคอมโพเนนต์นั้นเหมาะสมเก็บไว้ที่นั่น นั่นคือเหตุผลว่าฟังก์ชั่นที่สร้าง higher-order components จะถูกเก็บไว้ในไฟล์เดียวกับ view ถ้าเราใช้วิธีการแบบเดียวกันแต่ไม่ส่ง store ไป หรือกล่าวอีกนัยหนึ่งฟังก์ชั่นจะรับเฉพาะ consumer เท่านั้น และฟังก์ชันนี้จะถูกเรียกทุกครั้งเมื่อ store มีการเปลี่ยนแปลง

ถึงตอนนี้การใช้งานของเรามีเพียงการโต้ตอบกับ store เท่านั้นใน method `register`

```js
register: function (store) {
  if (!store || !store.update) {
    throw new Error('You should provide a store that has an `update` method.');
  } else {
    this._stores.push({ store: store });
  }
}
```

โดยใช้ `register` เราจะอ้างอิงไปยัง store ภายใน dispatcher อย่างไรก็ตาม `register` จะไม่มีการส่งข้อมูลอะไรกลับไป หรือเราอาจจะให้มันส่ง **subscriber** ที่รับฟังก์ชั่น consumer กลับไป

![Fluxiny - connect store and view](./fluxiny_store_view.jpg)

ผมเลือกส่งทั้ง store ไปยังฟังก์ชัน consumer แทนที่จะส่งแค่ข้อมูลที่เก็บไว้นนั้น เช่นเดียวกันในรูปแบบ higher-order components ส่วน view ควรใช้ getter ของ store เพื่อเรียกสิ่งที่ต้องการ ทำให้ใช้ store ค่อนข้างง่ายและไม่มีร่องรอยของ presentational logic

นี่คือ method register ที่เปลี่ยนแปลง:

```js
register: function (store) {
  if (!store || !store.update) {
    throw new Error(
      'You should provide a store that has an `update` method.'
    );
  } else {
    var consumers = [];
    var subscribe = function (consumer) {
      consumers.push(consumer);
    };

    this._stores.push({ store: store });
    return subscribe;
  }
  return false;
}
```

สิ่งสุดท้ายที่จะทำให้เสร็จสมบูรณ์คือ ทำให้ store แจ้งเมื่อข้อมูลมีการเปลี่ยนแปลง เราได้รวบรวมฟังก์ชัน consumer ไว้แล้ว แต่ตอนนี้ยังไม่มีการใช้งาน

ตามหลักการพื้นฐานของสถาปัตยกรรม Flux นั้น store จะเปลี่ยนสถานะเพื่อตอบสนองต่อ actions ใน method `update` เราจะส่ง `action` แต่เรายังสามารถส่งฟังก์ชัน `change` การเรียกฟังก์ชันนี้เพื่อให้เกิดการทำงานขึ้นที่ consumer:

```js
register: function (store) {
  if (!store || !store.update) {
    throw new Error(
      'You should provide a store that has an `update` method.'
    );
  } else {
    var consumers = [];
    var change = function () {
      consumers.forEach(function (l) {
        l(store);
      });
    };
    var subscribe = function (consumer) {
      consumers.push(consumer);
    };

    this._stores.push({ store: store, change: change });
    return subscribe;
  }
  return false;
},
dispatch: function (action) {
  if (this._stores.length > 0) {
    this._stores.forEach(function (entry) {
      entry.store.update(action, entry.change);
    });
  }
}
```

*ถ้าหากว่าเรา push `change` กับ `store` ใน array `_stores` หลังจากนั้นให้ `dispatch` เรียก method `update` และส่ง `action` กับ `change` ไปในฟังก์ชัน*

การใช้งานทั่วไป เพื่อสร้าง view และกำหนดสถานะเริ่มต้นของ store ในบริบทของการใช้งานของเราหมายถึง consumer จะถูกเรียกใช้อย่างน้อยหนึ่งครั้งเมื่อเรียกใช้งาน library ซึ่งสามารถทำได้อย่างง่ายใน method `subscribe`:

```js
var subscribe = function (consumer, noInit) {
  consumers.push(consumer);
  !noInit ? consumer(store) : null;
};
```

แน่นอนว่าบางครั้งเราไม่จำเป็นที่จะต้องกำหนดตัวแปร flag ขึ้นมาให้เป็นค่า false และนี่ก็เป็นหน้าตา dispatcher ที่เสร็จแล้ว:

<span class="new-page"></span>

```js
var Dispatcher = function () {
  return {
    _stores: [],
    register: function (store) {
      if (!store || !store.update) {
        throw new Error(
          'You should provide a store that has an `update` method'
        );
      } else {
        var consumers = [];
        var change = function () {
          consumers.forEach(function (l) {
            l(store);
          });
        };
        var subscribe = function (consumer, noInit) {
          consumers.push(consumer);
          !noInit ? consumer(store) : null;
        };

        this._stores.push({ store: store, change: change });
        return subscribe;
      }
      return false;
    },
    dispatch: function (action) {
      if (this._stores.length > 0) {
        this._stores.forEach(function (entry) {
          entry.store.update(action, entry.change);
        });
      }
    }
  }
};
```

<span class="new-page"></span>

## Actions

คุณอาจจะเห็นว่าเรายังไม่ได้พูดถึง action แล้ว actions คืออะไร? โดยทั่วไปมันเป็นแค่ object ที่มีเพียง `type` และ `payload`:

```js
{
  type: 'USER_LOGIN_REQUEST',
  payload: {
    username: '...',
    password: '...'
  }
}
```

`type` จะเป็นตัวบอกว่า action นั้นทำอะไร ส่วน payload จะเก็บข้อมูลที่เกี่ยวกับ event บางกรณีไม่จำเป็นต้องมีก็ได้

สิ่งที่น่าสนใจคือ `type` ที่ได้กล่าวไปแล้วในตอนต้น และที่สำคัญคือเราควรจะรู้ว่าต้องมี action อะไรในแอปพลิเคชันของเราบ้าง และตัวไหนที่คอยทำการ dispatch ไปยัง store ดังนั้นเราจึงสามารถประยุกต์ใช้[บางส่วน](http://krasimirtsonev.com/blog/article/a-story-about-currying-bind)ได้ และหลีกเลี่ยงการส่ง object action ตัวอย่างเช่น

```js
var createAction = function (type) {
  if (!type) {
    throw new Error('Please, provide action\'s type.');
  } else {
    return function (payload) {
      return dispatcher.dispatch({
        type: type,
        payload: payload
      });
    }
  }
}
```

`createAction` มีข้อดีดังนี้:

* เราไม่จำเป็นต้องจำ type ของ action อีกต่อไป ตอนนี้เรามีฟังก์ชันที่เราเรียกผ่าน payload เท่านั้น
* เราไม่จำเป็นต้องเข้าถึง dispatcher อีกต่อไปซึ่งเป็นประโยชน์อย่างมาก มิเช่นนั้นคุณต้องพิจารณาวิธีส่งข้อมูลทุกครั้งที่ใช้งาน dispatch
* ข้อสุดท้ายเราไม่ต้องจัดการ object อีกต่อไป เราแค่เรียกฟังก์ชันซึ่งดีกว่ามาก ซึ่งเป็น *static* objects ขณะที่ฟังก์ชันอธิบายถึง *process*

![Fluxiny actions creators](./fluxiny_action_creator.jpg)

วิธีการสร้าง actions ด้วยวิธีนี้นี้เป็นที่นิยมมาก ฟังก์ชั่นด้านบนเรียกว่า *action creators*.

## โค้ดที่สมบูรณ์

ในส่วนก่อนหน้า dispatcher ถูกซ่อนอยู่ในขณะที่เราดำเนินการ action เราอาจดำเนินการอีกครั้งเพื่อทำการ registry store:

```js
var createSubscriber = function (store) {
  return dispatcher.register(store);
}
```

และแทนที่จะ export dispatcher เราอาจ export เฉพาะสองฟังก์ชันนี้คือ `createAction` และ `createSubscriber` นี่คือโค้ดที่สมบูรณ์

```js
var Dispatcher = function () {
  return {
    _stores: [],
    register: function (store) {
      if (!store || !store.update) {
        throw new Error(
          'You should provide a store that has an `update` method'
        );
      } else {
        var consumers = [];
        var change = function () {
          consumers.forEach(function (l) {
            l(store);
          });
        };
        var subscribe = function (consumer, noInit) {
          consumers.push(consumer);
          !noInit ? consumer(store) : null;
        };

        this._stores.push({ store: store, change: change });
        return subscribe;
      }
      return false;
    },
    dispatch: function (action) {
      if (this._stores.length > 0) {
        this._stores.forEach(function (entry) {
          entry.store.update(action, entry.change);
        });
      }
    }
  }
};

module.exports = {
  create: function () {
    var dispatcher = Dispatcher();

    return {
      createAction: function (type) {
        if (!type) {
          throw new Error('Please, provide action\'s type.');
        } else {
          return function (payload) {
            return dispatcher.dispatch({
              type: type,
              payload: payload
            });
          }
        }
      },
      createSubscriber: function (store) {
        return dispatcher.register(store);
      }
    }
  }
};

```

ถ้าเราเพิ่มการสนับสนุน AMD, CommonJS และการใช้งาน global โค้ดที่สมบูรณ์จะมีทั้งหมด 66 บรรทัด ขนาดไฟล์ 1.7KB และถ้าบีบอัดเหลือ 795 bytes โดยการ minifying JavaScript

## Wrapper

เรามีโมดูล helper สองอันสำหรับการสร้าง Flux เราจะลองเขียนแอปนับจำนวนแบบง่าย ๆ โดยที่ไม่ใช้ React เพื่อให้เราเห็นรูปแบบการทำงาน

<span class="new-page"></span>

### Markup

เราจำเป็นต้องมี UI เพื่อโต้ตอบกับข้อมูลดังกล่าว:

```html
<div id="counter">
  <span></span>
  <button>increase</button>
  <button>decrease</button>
</div>
```

`span` ใช้เพื่อแสดงค่าจำนวนปัจจุบัน เมื่อมีการคลิกปุ่มจะเปลี่ยนจำนวนค่า

### View

```js
const View = function (subscribeToStore, increase, decrease) {
  var value = null;
  var el = document.querySelector('#counter');
  var display = el.querySelector('span');
  var [ increaseBtn, decreaseBtn ] =
    Array.from(el.querySelectorAll('button'));

  var render = () => display.innerHTML = value;
  var updateState = (store) => value = store.getValue();

  subscribeToStore([updateState, render]);

  increaseBtn.addEventListener('click', increase);
  decreaseBtn.addEventListener('click', decrease);
};
```

view จะได้รับฟังก์ชั่น store subscriber และ action สำหรับการ เพิ่มค่า / ลดค่า ในบรรทัดแรก ๆ นั้นเป็นเพียงแค่การ fetch DOM elements

หลังจากนั้นเราได้กำหนดฟังก์ชัน `render` ซึ่งมีหน้าที่ในการแสดงค่าลงในแท็ก span และฟังก์ชัน `updateState` ซึ่งจะถูกเรียกใช้เมื่อ store มีการเปลี่ยนแปลง เราได้ส่งสองฟังก์ชั่นนั้นไปใน `subscribeToStore` เนื่องจากเราต้องการให้ view มีการเปลี่ยนแปลงข้อมูล จำไว้ว่าฟังก์ชั่น consumer จะถูกเรียกอย่างน้อยหนึ่งครั้ง

สิ่งสุดท้ายที่ต้องทำคือผูก action กับ button element

### Store

ทุก action จะมี type การประกาศให้เป็นตัวแปรค่าคงที่วิธีที่ดีที่สุด เนื่องจากเราไม่ต้องมีการประมวลผลกับมัน

```js
const INCREASE = 'INCREASE';
const DECREASE = 'DECREASE';
```

โดยปกติเรามักจะมีเพียงหนึ่ง store เท่านั้น เพื่อความง่ายเรา store ให้มีเพียงอันเดียว

```js
const CounterStore = {
  _data: { value: 0 },
  getValue: function () {
    return this._data.value;
  },
  update: function (action, change) {
    if (action.type === INCREASE) {
      this._data.value += 1;
    } else if (action.type === DECREASE) {
      this._data.value -= 1;
    }
    change();
  }
};
```

`_data` คือ state ที่อยู่ใน store ส่วน `update` นั้นอย่างที่เราได้ทราบกันไปแล้วว่าจะถูกเรียกโดย dispatcher และเราจะ process action ภายในนั้น และฟังก์ชัน `change()` จะถูกเรียกเมื่อข้อมูลได้มีการเปลี่ยนแปลง ส่วน`getValue` นั้นเป็น public method จะถูกเรียกโดย view เมื่อต้องการเรียกข้อมูล ในกรณีนี้เป็นเพียงแค่ข้อมูลจำนวนนับ

### เชื่อมทุกส่วนเข้าด้วยกัน
ตอนนี้เรามี store ที่รอ action จาก the dispatcher และได้สร้าง view ไว้แล้ว เหลือเพียงสร้าง store subscriber มาเริ่มทำให้มันทำงานได้กันเถอะ

```js
const { createAction, createSubscriber } = Fluxiny.create();
const counterStoreSubscriber = createSubscriber(CounterStore);
const actions = {
  increase: createAction(INCREASE),
  decrease: createAction(DECREASE)
};

View(counterStoreSubscriber, actions.increase, actions.decrease);
```

และตอนนี้ view ได้ subscribe store เรียบร้อยแล้ว และจะ render โดย default เพราะ method `render` เป็นหนึ่งใน consumer

### Live demo

สามารถดูตัวอย่าง live demo ที่เว็บ JSBin [http://jsbin.com/koxidu/embed?js,output](http://jsbin.com/koxidu/embed?js,output) 
ถ้าคุณคิดว่าตัวอย่างนี้ยังไม่เพียงพอสามารถไปดูตัวอย่างเพิ่มเติมใน repository Fluxiny [https://github.com/krasimir/fluxiny/tree/master/example](https://github.com/krasimir/fluxiny/tree/master/example) ซึ่งได้ใช้ React ในการสร้าง view

*การใช้ Flux ที่กล่าวถึงในบทนี้จะอยู่ที่นี่ [github.com/krasimir/fluxiny](https://github.com/krasimir/fluxiny) สามารถใช้งานได้[โดยตรง](https://github.com/krasimir/fluxiny/tree/master/lib)ในเบราว์เซอร์ หรือผ่าน [npm dependency](https://www.npmjs.com/package/fluxiny)*
