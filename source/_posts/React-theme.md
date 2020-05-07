---
title: Thiết lập theme cho React app với Context API và SASS
catalog: true
date: 2020-05-07 11:22:42
subtitle:
header-img:
tags:
  - react
  - react-theme
  - react-context
  - sass
---

## Dark mode >< Light mode

Ngày nay người người dùng dark mode, nhà nhà dùng dark mode, dark mode khắp mọi nơi.

Mình không phải fan của dark mode, nhưng không có nghĩa là mình có thể mặc kệ không đưa nó vào để người dùng thể tận hưởng sở thích của họ.

Có rất nhiều cách để đưa dark mode vào React app, ở bài viết này mình xin chọn cách sử dụng Context API và SASS.

Mình không chắc đây là cách tốt nhất, hoặc có lẽ là cách tốt, nhưng chí ít hiện tại nó chạy được với dự án của mình mà không có vấn đề gì nảy sinh.

Trước khi vào bài thì bạn cần có một chút căn bản về [Context API](https://reactjs.org/docs/context.html) và [SASS](https://sass-lang.com/).

Ngoài ra thì chúng ta sẽ sử dụng [Hook](https://reactjs.org/docs/hooks-intro.html) thay cho Class Component.

*OK chúng ta bắt đầu thôi*

## Context

Đầu tiên chúng ta tạo file chứa context.

```js
// themeContext.js

import React from 'react';

export const themes = {
  light: 'light',
  dark: 'dark'
};

const ThemeContext = React.createContext({
  theme: themes.light, // theme value
  setTheme: () => {} // function to change theme
});

export default ThemeContext;
```

Nội dung file bao gồm biến `themes` chứa 2 định nghĩa theme: `light` và `dark`.

Sau đó export một context chứa giá trị `theme` cùng với hàm để thay đổi giá trị này. Giá trị `theme` có mặc định là `themes.light`.

## ContextProvider

Đưa theme context provider vào App, hoặc tại một root nào đó mà bạn muốn theme của chúng ta có ảnh hưởng tới toàn bộ cây phía dưới.

```js
// App.js

import React, { useState } from 'react';
import Layout from './layout/Layout';
import ThemeContext, { themes } from './themesContext';

const App = () => {
  /*
    this state is actually where the theme data is store
    we will then pass this state and it accompany function to ThemeContext
  */
  const [theme, setTheme] = useState(themes.light);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Layout />
    </ThemeContext.Provider>
  )
}
```

Như vậy, toàn bộ cây con của `Layout` sẽ có `theme` context sẵn sàng để sử dụng.


## Use Context

Sử dụng theme ở các Component

```js
// layout/Layout.js

import React, { useContext } from 'react';
import './Layout.scss'; // normal styles go here
import './Layout.dark.scss'; // styles on dark mode go here

export default function Layout() {
  // useContext hook, we only get theme value
  const { theme } = useContext(ThemeContext);
  
  return (
    <div className={theme}> // the theme value is using here as class
      <div className="header">
        {..someStuff}
      </div>
      <div className="middle">
        {..someOtherStuff}
      </div>
    </div>
  )
}
```

Chúng ta truy cập giá trị của `theme` context bằng `useContext` hook, sau đó dùng giá trị này để làm className cho component.

Bạn thấy rằng Component này import 2 SASS file: `Layout.scss` và `Layout.dark.scss`, kết hợp với `theme` className, cách chúng ta tổ chức style như sau:

* Tất cả style cần thiết và style light mặc định để vào `Layout.scss`
* Style cho dark mode để vào `Layout.scss`, với wrapper `.dark` bao toàn bộ.

Như vậy khi giá trị của `theme` context trở thành `dark`, các style cho dark mode sẽ được kích hoạt.

```scss
/* Layout.scss */

.header {
  /* all CSS that the page need for normal render */
  background: #fff;
  color: #000;
}

.middle {
  /* other CSS for difference class */
}
```

```scss
/* Layout.dark.scss */

.dark {
  .header {
    background: #000;
    color: #fff;
  }

  .middle {
    /* blah blah */
  }
}
```

Trong ví dụ này, mặc định `.header` sẽ có nền trắng, chữ đen. Khi dark mode được bật sẽ thay đổi thành nền đen chữ trắng.

Style trong ví dụ chỉ để cho bạn thấy ý tưởng của hướng làm này, dĩ nhiên app thật sự sẽ có style phức tạp hơn.


## Update Context 

Từ từ đã, làm thế nào để chúng ta đổi được theme ?

Hãy tạo một nút `ThemeButton` như sau

```js
// ThemeButton.js

import React, { useContext } from 'react';
import ThemeContext, { themes } from 'themeContext.js';

export default function ThemeButton() {
  /* you might notice here, unlike Layout.js need only theme value
    we get both theme value and setTheme function this time
    because Layout.js only use theme value, and here we need to change it
  */
  const { theme, setTheme } = useContext(ThemeContext);

  // for convenient we get other theme value here, because we have only 2 themes
  const otherTheme = Object.values(themes).find(value => value !== theme);

  const toggleTheme = () => {
    setTheme(otherTheme);
  };

  return (
    <button
      type="button"
      className="themeSwitcher"
      onClick={toggleTheme} 
    >
      Switch {otherTheme} theme
    </button>
  )
}
```

Sử dụng `useContext` giống như ở `Layout`, nhưng lần này chúng ta truy cập cả giá trị lẫn hàm điều khiển context.

Khi người dùng click vào Button, chúng ta dùng hàm điểu khiển để thay đổi theme hiện tại.

Bay giờ bạn có thể đặt `ThemeButton` ở nơi nào đó thuận tiện cho người dùng.

## More about Context

Hiểu thêm về context value/control function

Quay trở lại Context file một chút

```js
// themeContext.js
.
.
.
const ThemeContext = React.createContext({
  theme: themes.light, // theme value
  setTheme: () => {} // function to change theme
});

```

Khi export Context chúng ta trả về một Object có 2 thành viên, `theme` và `setTheme`.

`theme` là giá trị của context, bất cứ component nào bao bởi ContextProvider đều có thể truy suất giá trị này.

`setTheme` là hàm để điều khiển giá trị của context.

Và tại App.js, khi chúng ta khởi tạo ContextProvider

```js
// App.js
.
.
.
  const [theme, setTheme] = useState(themes.light);

  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      <Layout />
    </ThemeContext.Provider>
  )
```

Ở đây chúng ta cung cấp 1 cặp state và setState hook vào cho context, như vậy bên dưới cây component, giá trị của context cùng với hàm điều khiển thực ra chính là cặp state, setState này.

Khi chúng ta gọi `setTheme`, context sẽ dùng setState tại App.js để thay đổi `theme` state, context sau đó cập nhật thay đổi tới các consumer.

## Kết

Vậy là xong, từ bây giờ khi tạo component mới, bạn chỉ cần tạo kèm 2 file SASS, 1 cho light theme và 1 cho dark theme vào wrap bới `.dark` class.
Bất cứ component nào được sử dụng bên dưới `Layout` đều sẽ chịu ảnh hưởng của `.dark` class này, bạn không cần thiết phải truy cập tới theme context để xét giá trị của nó nữa.

Hi vọng cách làm này có thể giúp bạn có thêm kiến thức về context và cách thiết lập theme cho React app.