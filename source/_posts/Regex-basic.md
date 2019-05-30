---
title: Regex basic
catalog: true
date: 2018-06-11 15:22:42
subtitle:
header-img:
tags: regex
---

## Điều gì khiến bạn quan tâm tới Regex

Regex về cơ bản là một string, `abcd` là một regex, `^\d\d+.*\w+` cũng là một regex.

Regex mô tả một **mẫu** dùng để tìm kiếm đoạn text hoặc một vị trí nào đó trong một đoạn văn bản - có thể ngắn hoặc dài.

Regex thông thường có bốn công dụng chính: **tìm kiếm** trong một lượng văn bản lớn, **thay thế** hoặc **chèn** đoạn văn bản vào một vị trí nào đó, **kiểm tra** tính đúng đắn của một đoạn văn bản, hoặc để **chia** văn bản hoặc câu thành những phần có cấu trúc định sắn.

Điều gì làm regex khác biệt với cách tìm kiếm khác ? Đó là khả năng mô tả những mẫu phức tạp.

Ví dụ khi bạn muốn tìm kiếm tất cả những tệp có phần mở rộng .txt, chúng ta có thể sử dụng lệnh find với wildcard như sau

`find . -name "*.txt"`

Nhưng kết quả ra quá nhiều, và bạn nhớ thêm rằng tệp mình cần tìm có lẽ chứa từ **system** trong đó, vậy phải làm thế nảo ?

Trong trường hợp này regex có thể giúp bạn với một pattern đơn giản

`find -regex ".*system.*\.txt"`

Kết quả thu được sẽ chỉ bao gồm những tệp có phần mở rộng .txt và chứa từ system trong tên. Lúc này có thể bạn chưa hiểu những `.*\` đóng vai trò gì ở đây, nhưng chỉ cần dành một chút thời gian tìm hiểu bạn sẽ thấy regex không hề khó học nếu như bạn đã từng nghĩ thế.

Một ví dụ khác về sự mạnh mẽ của regex trong việc thay thế và xứ lý văn bản như sau

Bạn có một đoạn code chạy trên ngôn ngữ Java và muốn chuyển sang dùng cho Scala, tuy nhiên cú pháp của 2 ngôn ngữ có phần khác biệt

Ngôn ngữ Java có cú pháp như sau: `<type> <name> = <value>`

Còn Scala chúng ta dùng: `val <name>: <type> = <value>`

Ví dụ về syntax của Java

```
String s1 = "Computer Science";
int x = 307;
String s2 = s1 + " " + x;
String s3 = s2.substring(10,17);
String s4 = "is fun";
String s5 = s2 + s4;
```

Làm thế nào để có thể chuyển code của Java về Scala ? Chúng ta có thể Ctrl+X `<type>`, Ctrl+V nó về phía sau `<name>`, sau đó gõ thêm dấu `:`

Và sẽ mất khoảng 10 phút cho việc copy paste với 100 dòng code

Nếu chúng ta biết regex chúng ta sẽ không phải vất vả như vậy, chỉ cần mở tool replace của editor lên và gõ

`(\w+)\s(\w+)\s=`

vào ô tìm kiếm, sau đó gõ

`val $2: $1 =`

vào ô thay thế, bật mode regex lên và ấn **Enter**, đoạn code trên sẽ ngay lập thức thay đổi

```
val s1: String = "Computer Science";
val x: int = 307;
val s2: String = s1 + " " + x;
val s3: String = s2.substring(10,17);
val s4: String = "is fun";
val s5: String = s2 + s4;
```

và với 100 hay 1000 LOC thì regex của chúng ta vẫn có thể xử lý êm đẹp.

Những ví dụ trên chúng ta có thể gặp rất nhiều trong công việc thường ngày, và những pattern đó có thể được viết ra hoàn toàn với kiến thức rất cơ bản của regex.

Làm quen với regex các bạn sẽ tìm thêm rất nhiều ứng dụng của nó trong công việc, nhất là đối với một lập trình viên. Bởi vậy đừng chờ đợi nữa, hãy bắt đầu học regex ngay hôm nay.

## Những kiến thức đầu tiên về Regex

*Note: regex được viết giữa hai dấu sượt phải `/../`, chỉ thị `g` ở cuối tức tìm trên toàn bộ chuỗi mẫu*

### Các ký tự đơn giản

Đầu tiên là regex có thể được sử dụng như một phép tìm kiếm thông thường, chúng ta có thể tìm kiếm hầu hết các ký tự bằng cách gõ chúng vào regex (trừ một vài ký tự đặc biết chúng ta sẽ tìm hiểu tiếp theo).
 
VD: tìm tất cả các lần xuất hiện của từ **matching**


`/matching/g`


>Here is some text to try **matching**. Type any word from this text and it will get matched and highlighted.

### Chữ cái và chữ số

Trong số các ký tự đặc biệt, `\w`  và `\d` được sử dụng phổ biến nhất. `\w\` đại diện cho tất cả các chữ cái hoặc chữ số latin, `\d` đại diện cho tất cả chữ số thập phân.

VD: tìm tất cả các chữ số và chữ cái


`/\w/g`


> **|H|e|r|e| |a|r|e| |s|o|m|e| |w|o|r|d|s|**: **|o|n|e|**, **|t|w|o|**, **|t|h|r|e|e|**, **|F|O|U|R|**, **|1|2|3|**, **|4|5|6|**


`/\d/g`


> Here are some words: one, two, three, FOUR, **|1|2|3|**, **|4|5|6|**

### Ký tự lặp

`\w` và `\d` ở trên tìm các ký tự một cách tách biệt, nghĩa là đối với từ **Here**, thực chất regex sẽ match tương ứng với **H**, **e**, **r** và **e**, tương tự **123** sẽ là **1**, **2**, **3**.

Để có thể match toàn bộ từ hoặc số có nhiều chữ số ta sẽ dùng ký tự `*` hoặc `+`

`+` sẽ match ký tự trước nó xuất hiện 1 hoặc nhiều lần.

`*` sẽ match ký tự trước nó xuất hiện 0 hoặc nhiều lần.


`/\w+/g`


> **|Here| |are| |some| |words|**: **|one|**, **|two|**, **|three|**, **|FOUR|**, **|123|**, **|456|**

Khác với khi không có `+`, ở đây regex sẽ match toàn bộ từ và số chứ không match riêng lẻ từng ký tự.

### Tập ký tự

Chúng ta có thể tìm trong một tập ký tự giới hạn bằng cách cho chúng vào ngoặc vuông `[]`

Ví dụ: chỉ match những ký tự a hoặc b hoặc c `[abc]`, từ a đến c `[a-c]`, từ 0 đến 5 `[0-5]`, chỉ ký tự hoa `[A-Z]`


`/[a-h]/g`


> H**e**r**e** **a**r**e** som**e** wor**d**s: on**e**, two, t**h**r**ee**, FOUR, 123, 456

### Ký tự đặc biệt

Để sử dụng các ký tự đặc biệt trong regex chúng ta cần thêm `\` ở phía trước chúng: `\.`, `\(`, `\^` ...


`/[\+\\]/g`


> Some special symbols: [^.**\\\+**\*]

### Ký tự thay thế

Cú pháp `a|b` sẽ match a hoặc b, ta có thể dùng `|` cho điều kiện phức tạp hơn như: `[a-f]|\d` sẽ match chữ cái từ a đến f **hoặc** chữ số


`/[a-f]+|\d+/g`


> Wor**d**s: on**e**, two, thr**ee**, FOUR. An**d** num**be**rs: **1**, **2**, **3**, **4**, **100**, **200**, **300**, **400**.

### Match bất cứ ký tự nào và ký tự tùy chọn

Dấu `.` sẽ match tất cả ký tự

Đặt `.*` sẽ match với chuỗi rỗng

Đặt `.+` sẽ match toàn bộ dòng

Đặt `?` sau ký tự sẽ biến ký tự đó thành tùy chọn - có thể có hoặc không. Ví dụ: `s?ing` sẽ match với **sing** và với phần **ing** của từ **king**


`/s?ing/g`


> The K**ing** is **sing|ing**

### Dòng, từ và khoẳng trắng

Ký tự `^` match với vị trí bắt đầu dòng và `$` tương ứng kết thúc dòng.

Ví dụ: `^\w+` sẽ match từ đầu tiên của dòng và tương ứng `\w+$` sẽ match từ cuối dòng

Ký tự `\b` sẽ match với giới hạn từ

Ví dụ: `\w\b` sẽ tìm tất cả ký tự đứng đầu của từ, `\b\w` sẽ tìm ký tự đứng cuối từ

Ký tự `\s` match với 1 khoảng trắng, để match một hay nhiều khoảng trắng ta có thể dùng `\s+`

### Số lần lặp

Chúng ta đã biết `+` và `*` cho ta lựa chọn match với ký tự lặp 0, 1 hoặc nhiều lần. Tuy nhiên việc giới hạn số lần lặp có thể xảy ra.

Để làm điều này chúng ta sử dụng cặp ngoặc nhọn `{}` phía sau ký tự:

- `{1, 4}` lặp từ 1 đến 4 lần

- `{4,}` lặp tối thiểu 4 lần

- `{,4}` lặp tối đa 4 lần

- `{4}` lặp đúng 5 lần

Match các từ có 4 ký tự trở lên


`/\b\w{4,}\b/g`


> It is a **long** **established** **fact** **that** a **reader** **will** be **distracted** by the **readable** **content** of a **page** **when** **looking** at its **layout**

### Nhóm và thay thế

Ta có thể nhóm kết quả từ regex bằng cặp ngoặc tròn `()`, sau đó có thể dùng lại nhóm trong regex bằng cú pháp `\1` `\2` với 1, 2 là thứ tự xuất hiện của nhóm trong regex từ trái qua phải

Ví dụ: `(\w+)\s\1` có thể dùng để match các từ bị lặp lại cách nhau bởi 1 dấu cách


`/(\w+)\s\1/g`


> **me me** **you you** he she it

Ứng dụng quan trọng của việc nhóm là ta có thể sử dụng nhóm để thay thế với cú pháp `$1` `$2`, tương tự như trên 1 2 là thứ tự của nhóm.

Ví dụ: chuyển đổi format `mm-dd-yyyy` thành `yyyy-mm-dd`


`/(\d{2})-(\d{2})-(\d{4})/g replace with $3-$1-$2`


> 02-20-1994 -> 1994-02-20

### Lookaheads

Khi muốn match ký tự với điều kiện theo sau nó là một ký tự hoặc chuỗi nào đó ta sử dụng cú pháp `(?=)`

Ví dụ: match ký tự đứng ngay trước dấm chấm `.`


`/\w(?=\.)/g`


> It is a long established fact tha**t**. A reader will be distracted by the readable content of a pag**e**. When looking at its layou**t**. The point of using Lorem Ipsum is that it has a more-or-less norma**l**. Distribution of letters, as opposed to using 'Content here, content here', making it look like readable Englis**h**.

Ngược lại nếu muốn match ký tự không theo sau bởi gì đó ta dùng `(?!)`

Ví dụ: match các từ không đứng trước dấu chấm `.`


`\w+(?!\.)\b`


>**It is a long established fact** that. **A reader will be distracted by the readable content of a** page. **When looking at its** layout. **The point of using Lorem Ipsum is that it has a more-or-less** normal. **Distribution of letters, as opposed to using 'Content here, content here', making it look like readable** English.

## Ví dụ đơn giản làm việc với regex

Bài toán: regex validate email, thỏa mãn các yêu cầu sau
- Phải bắt đầu bằng chữ cái
- Chỉ bao gồm chữ cái, chữ số và 3 ký tự `- _ .`
- Không chứa ký tự in hoa

Ngoài ra các yêu cầu cơ bản của email là:
- Gồm 2 phần phân cách bởi dấu `@`
- Phần tên miền chỉ bao gồm chữ cái, chữ số hoặc dấu gạch ngang `-`
- Kết thúc bằng top-level domain

Chúng ta tiến hành phân tích và viết regex:

Bắt đầu bằng chữ cái và không có chữ in hoa
- `^[a-z]`

Phần còn lại của địa chỉ có thể chứa nhiều chữ cái, chữ số và chỉ các ký tự `- _ .`
- `[a-z0-9-_.]+`

Kết thúc phần địa chỉ với ký tự `@`
- `@`

Tên miền chỉ bao gồm chữ cái, chữ số và ký tự `-`, tuy nhiên tên miền có thể có nhiều lớp nên cần thêm ký tự `.`
- `[a-z0-9-.]+`

Kết thúc bằng top-level domain: bắt đầu bằng dấu `.`, tiếp theo chỉ bao gồm chữ cái, cần ít nhất 2 ký tự trở lên, và là phần kết thúc của chuỗi
- `\.[a-z]{2,}$`

Như vậy kết hợp các thành phần lại với nhau ta được regex hoàn chỉnh


`/^[a-z][a-z0-9-_.]+@[a-z0-9-.]+\.[a-z]{2,}$/g`



>**thanh@gmail.com (matched)**
>
>**septeni69@septeni-technology.jp (matched)**
>
>**valid.email@lx.us.com (matched)**
>
>space is not allowed@gmail.com (not-matched)
>
>1startwithnumber@mail.com (not-matched)
>
>not_contains_at_symbol.com (not-matched)
>
>domainiswrong@hotmail (not-matched)
>
>UpperCaseIsWrong@gmail.com (not-matched)


Trên đây là giới thiệu cơ bản về regex, hi vọng có thể giúp các bạn tiếp cận được ý tưởng về regex ở mức khởi đầu.


### Tham khảo
https://www.rexegg.com/

http://regextutorials.com/