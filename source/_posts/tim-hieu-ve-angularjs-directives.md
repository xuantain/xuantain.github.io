---
layout:     post
title:      "Tìm hiểu về AngularJS Directives (Phần 1)"
subtitle:   "Bài viết tìm hiểu về AngularJS Directives và cách tạo một Custom Directive - Phần 1 (Directives là gì? Cách tạo custom directive, restrict, template, templateUrl, priority)."
date:       2014-10-24
author:     "xuantain"
# catalog:    true
# header-img: ""
tags:
    - Angular
    - Angular 1
    - Javascript
---


### Directives là gì?

Các thuộc tính, id, class, name, onClick, … của một DOM element được gọi chung là directives. AngularJS sẽ dựa vào những directive đó để đính kèm các chỉ thị hoặc các sự kiện tới DOM element cần xử lý, thậm chí là thay đổi cấu trúc của DOM. 

AngularJS có sẵn một bộ directives: ng-app, ng-module, ng-controller, …  Để phát triển ứng dụng với AngularJS thì cần phải nắm rõ cách hoạt động và mục đích sử dụng của những directive đó. Trong phạm vi bài viết này sẽ không giải thích và hướng dẫn cách sử dụng những directive sẵn có của AngularJS.


### <span id="compileHTML">Cách Directives được biên dịch</span>
Khi một trang được load thì web browser sẽ tự động parses HTML vào DOM.
Việc biên dịch HTML được thực thi trong 3 bước:

  1. $compile sẽ quét qua các DOM elements, nếu compiler tìm thấy element nào có khai báo directive, thì directive tương ứng sẽ được thêm vào danh sách các directives phù hợp với DOM element. Một element có thể khai báo nhiều directives.
  2. Khi một DOM element khai báo nhiều directives, compiler sẽ sắp xếp thứ tự biên dịch các directives theo thuộc tính priority của mỗi directive. Compiler functions của mỗi directive sẽ được thực thi. Mỗi compiler function có thể làm thay đổi DOM. Mỗi compiler function phải trả về một link function, xử lý liên kết giữa DOM và scope.
  3. $compile liên kết template với scope bằng cách gọi linking function đã liên kết từ bước 2. Đăng ký theo dõi các elements và thiết lập cơ chế $watchs với scope của directive được cấu hình.

Sau khi thực thi biên dịch HTML thì giữa scope và DOM sẽ có sự ràng buộc (a live binding), mỗi một thay đổi ở model trên compiled scope sẽ được ánh xạ vào DOM.


### <span id="createD">Cách tạo Custom Directive</span>

Tên của Angular Directives được khai báo theo kiểu [camelCase](http://en.wikipedia.org/wiki/CamelCase).

Khi khai báo tên một directive thì tốt nhất bạn nên đặt thêm tiền tố, mục đích là trách bị trùng tên với các từ khoá của Angular hoặc của framework Javascript nào khác có sử dụng trong dự án. Thêm nữa, trong tương lai khi AngularJS nâng cấp version thì sẽ có bổ sung thêm các từ khóa có thể trùng với tên directive mà bạn đã đặt, bạn sẽ mất thời gian để chỉnh sửa lại. 

<ins>VD:</ins>

Khai báo directive

```js
  angular.module('demoModule', [])
    .directive('<span class="red">demoDirective</span>', function(){
    return {
      // TODO
    }
  });
```

Khi sử dụng

```js
  <p <span class="red">demo-directive</span> >
```


#### <em>restrict</em>

khai báo kiểu directive. Có các giá trị (A, E, C, M) tương ứng 4 kiểu directive:

- Kiểu <span class="green">a</span>ttribute (A)

```js
  <span class="blue"><div</span> <span class="red">directive</span><span class="blue">></span>
```

- Kiểu <span class="green">e</span>lement (E)

```js
  <span class="blue"><directive></directive></span>
```

- Kiểu <span class="green">c</span>lass (C)

```js
  <span class="blue"><div</span> class=<span class="red">”directive”</span><span class="blue">></span>
```

- Kiểu co<span class="green">m</span>ment (M)

```js
  <!-- directive: directive -->
```

<ins>VD:</ins><br />

```js
  <div ng-app="demoModule">
    <div ng-controller="demoController">
      <span class="red">
        <ANY demo-directive><any>
        <demo-directive></demo-directive>
        <ANY class="demo-directive"></any>
        <!-- directive: demo-directive -->
      </span>
    </div>
  </div>
```


#### <em id="template">template</em>

Định nghĩa template, phần hỉển thị nội dung cho directive của bạn (không sử dụng nếu dùng [templateUrl](#templateUrl)).

<ins>VD:</ins>

```js
  var myApp = angular.module('demoModule', []);
  myApp.directive('demoDirective', function(){
    return {
      restrict: "ACEM",
      <span class="red">template: "<h3>This is demo directive</h3>"</span>
    }
  });
```

#### <em id="templateUrl">templateUrl</em>

Khai báo đường dẫn (Url) của file định nghĩa template được dùng cho directive. Được sử dụng khi template của directive phức tạp, hoặc bạn muốn tách riêng phần định nghĩa template directive ra một file khác. Không sử dụng nếu dùng [template](#template), nếu không directive sẽ ưu tiên sử dụng nội dung trong template.

<ins>VD:</ins>

```js
  var myApp = angular.module('demoModule', []);
  myApp.directive('demoDirective', function(){
    return {
      restrict: "AE",
      <span class="red">templateUrl: "../templates/demoDirective.html"</span>
  }});
```


#### <em>priority</em>

Thứ tự thực thi directive.

Khi một DOM element khai báo nhiều directive thì cần phân định rõ thứ tự thực thi directive, vì vậy priority được dùng để sắp xếp các directive trước khi [compile function](#compile) được gọi. Priority được khai báo kiểu number, theo thứ tự thực thi 0 → 1 → 2 → … → n (Giá trị mặc định của priority là “0”). 

<ins>VD:</ins>

Có 3 directive

```js
  var myApp = angular.module('demoModule', []);
  myApp.directive('<span class="red">directive1</span>', function(){
    return {
      restrict: "A",
      template: "<h3>This is directive1</h3>",
      <span class="red">priority: 1</span>
  }});
  myApp.directive('<span class="red">directive2</span>', function(){
    return {
      restrict: "A",
      template: "<h3>This is directive2</h3>",
      <span class="red">priority: 2</span>
  }});
  myApp.directive('<span class="red">directive3</span>', function(){
    return {
      restrict: "A",
      template: "<h3>This is directive3</h3>",
      <span class="red">priority: 3</span>
  }});
```

Khai báo sử dụng

```js
  <span class="blue"><p</span> <span class="red">directive2 directive1 directive3</span> <span class="blue">></p></span>
```

Thứ tự thực thi

```js
  directive1 -> directive2 -> directive3
```

Trường hợp các directive có giá trị priority bị trùng nhau thì trình biên dịch sẽ thực thi directive được khai báo trước trong thẻ HTML.

<ins>VD:</ins>

Sử dụng 3 directive ở ví dụ trên, nhưng khai báo <span class="red">“priority: 1”</span> cho cả 3.

```js
  <span class="blue"><p</span> <span class="red">directive2 directive1 directive3</span> <span class="blue">></p></span>
```

Thứ tự thực thi

```js
  directive2 -> directive1 -> directive3
```

Các “pre-link” function cũng được thực thi theo thứ tự của priority, còn với “post-link” thì được thực thi theo thứ tự ngược lại. 

Nếu các directive đó đều có tác động đến việc hiển thị của DOM element, thì trình biên dịch sẽ ưu tiên xử lý của directive có giá trị priority nhỏ nhất.
