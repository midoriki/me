---
title: Regex replace
date: 2019-06-04 17:43:28
tags: regex
---

# Replace in Regex

Bên cạnh chức năng chính là tìm kiếm, thay thế văn bản là công dụng rất hữu ích của biểu thức chính quy, nó có thể giúp bạn tiết kiệm được rất nhiều thời gian và cả công sức nữa.

### Bắt đầu với một ví dụ đơn giản.

Bạn có một file CSV lưu trữ data kiểu số được xuất từ một nguồn nào đó, tuy nhiên lẫn trong những con số thông thường file này đôi khi xuất hiện số thập phân dạng string với ngoặc kép bao quanh

```csv
234 43 562 30 23 6 5 21 5 "34.3"
234 "87.32" 4 72 3 54 531 42 432 992
45 23 432 "3452.13" 322 43 20 "303" 8 33
.
.
.
```
Nhiệm vụ của bạn là trước khi xử lý file cần phải loại bỏ dấu ngoặc kép và phần sau dấu phẩy của các con số này.

Việc loại bỏ dấu ngoặc kép có vẻ rất dễ dàng, bạn chỉ cần tìm ký tự <"> và thay thế bằng khoảng trắng.

Vậy còn chữ số thập phân? Với tìm kiếm thông thường chúng ta chỉ có thể tìm được ký tự <.>, sau đó xóa tay phần số còn lại **cho từng số** mà chúng ta tìm được?

Không khả thi phải không nào, chỉ cần file có 100 dòng là mờ mắt rồi.

May mắn rằng chúng ta có Regex và công cụ thay thể của nó.

Mở file CSV bằng một trình editor nào đó có hỗ trợ tìm kiếm và thay thế bằng Regex (VSCode, Sublime, Notepad++, ...) và chọn **Find & Replace** (shortcut phổ biến là Ctrl+H trên Windows).

Chương trình sẽ cho bạn 2 ô input, ô đầu tiên để nhập pattern tìm kiếm, ô thứ 2 để nhập pattern thay thế.

Ví dụ giao diện box tìm kiếm thay thế của Visual Studio Code
<p align="center">
{% asset_img vscode-replace-box.jpg "VSCode replace toolbox" %}
</p>

Với đề bài trên, pattern tìm kiếm của chúng ta sẽ là:

```js
\"\d+\.?\d*\"
```
|          |                                  |
|----------|--------------------------------- |
| **\\"**  | tìm ký tự <"> đầu tiên           |
| **\\d+** | tìm 1 đến nhiều chữ số liền sau  |
| **\\.?** | tìm ký tự <.>                    |
| **\\d*** | tìm 0 đến nhiều chữ số tiếp theo |
| **\\"**  | tìm ký tự <"> cuối cùng          |

Pattern này đảm bảo sẽ match cả số có ngoặc kép tuy nhiên không chứa phần thập phân.

<p align="center">
{% asset_img vscode-find-sample-1.jpg "VSCode find sample 1" %}
</p>

Vậy làm thế nào để dùng pattern này cho việc thay thế ?

Chúng ta sẽ sử dụng **group**!

Khi một regex pattern được chia thành các group, chúng ta có thể sử dụng ký tự đại diện cho các group đó để điều khiển phần nào được giữ lại, thêm vào hoặc bỏ đi.

Ta sẽ group pattern trên như sau

```js
\"(\d+)\.?\d*\"
```

Với bài toán này chúng ta chỉ cần một group duy nhất, giữ lại phần nguyên của chuỗi, những phần còn lại đều là phần thừa.

Ký tự đại diện cho group là:
+ $<số thứ tự của group trong pattern, đánh số từ 1>

Như vậy chúng ta có thể tham chiếu đến phần nguyên của chuỗi cần thay thế bằng ký tự **$1**

Cuối cùng chúng ta chỉ cần điền pattern và ký tự thay thế vào thanh tìm kiếm và thay thế

<p align="center">
{% asset_img vscode-replace-sample-1.jpg "VSCode replace sample 1" %}
</p>

Chọn thay thế tất cả và ta được kết quả

<p align="center">
{% asset_img vscode-result-sample-1.jpg "VSCode result sample 1" %}
</p>

Như vậy chúng ta đã dùng regex pattern để tìm kiếm và thay thế thành công.

### Thực hành với một số ví dụ khác

1. Che đậy số điện thoại

Trong một file dữ liệu có chứa các số điện thoại có dạng **012-3456-7869**, tìm kiếm và thay đổi số điện thoại thành dạng **012-34\*\*-\*\*69**

Tuy nhiên chiều dài vùng thứ nhất dao động từ 2 đến 4 chữ số, vùng thứ hai từ 3 đến 4 chữ số.

Dữ liệu mẫu như sau

```
010-2104-8284
02-8545-8458
0134-058-9834
```

<details>
  <summary>Đáp án</summary>
  <p>

|              |                                    |
|--------------|------------------------------------|
|Pattern       | (\d{2,4}-\d{1,2})(\d{2}-\d{2})(\d+)|
|Replace string| $1\*\*-\*\*$3                          |

</p></details>

2. Thêm ngoặc tròn cho arrow function

Một ngày đẹp trời bạn nhìn lại code JS của mình và nhìn thấy hàm mình viết chưa đúng chuẩn e bi en bi, hàm một biến không vất vào trong ngoặc, đôi chỗ thừa thiếu space, bạn quyết định sửa lại cho bằng bạn bằng bè.

```js
const getSomething =name => {
  // get some thing with that name
}

const calculateSomething = value  => {
  // return something with that value
}

const seriousFunction = (data) => {
  // this function declaration is nice
}
```

<details>
  <summary>Đáp án</summary>
  <p>

|              |                     |
|--------------|---------------------|
|Pattern       | =\s*([^\(\s]+)\s*=> |
|Replace string| =\s($1)\s=>         |

</p></details>

### Kết

Hi vọng qua bài viết này bạn có thể nắm được ý tưởng cơ bản của việc sử dụng Regular Expression trong việc tìm kiếm thay thể văn bản. Có rất nhiều use case trong công việc hằng ngày và cả trong lập trình mà Regex có thể giúp chúng ta tiết kiệm thời gian và đôi khi là yếu tố thiết yếu để làm công việc đó.

Nếu muốn thực hành với nhiều ví dụ hơn bạn có thể ghé qua:

+ [RegexTutorials.com](http://regextutorials.com)
+ [My RegexpPractice page](/regexp)