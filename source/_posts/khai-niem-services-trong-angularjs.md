---
layout:     post
title:      "Khái niệm services trong AngularJS"
subtitle:   "Đây là bài viết tìm hiểu về khái niệm Services trong AngularJS"
date:       2014-12-20
author:     "xuantain"
# catalog:    true
# header-img: ""
tags:
    - Angular
    - Angular 1
    - Javascript
---


## Angular Services là gì?

> Angular Services là những đối tượng [singleton](http://en.wikipedia.org/wiki/Singleton_pattern), cung cấp các phương thức (method) liên quan đến một chức năng cụ thể, được dùng chung và chỉ được khởi tạo một lần duy nhất trong một ứng dụng Angular.

- Một Service chỉ được khởi tạo khi có thành phần trong ứng dụng phụ thuộc vào nó.
- Mỗi thành phần trong ứng dụng phụ thuộc vào một Service nào đó, thì chỉ nhận được tham chiếu đến duy nhất một đối tượng thể hiện của Service đó.

> Các thành phần trong Angular có thể trao đổi, chia sẻ dữ liệu với nhau thông qua Service.

Angular cung cấp sẵn một số Service để xây dựng ứng dụng:

| $anchorScroll | $exceptionHandler | $locale | $q |
| - | - | - | - |
| $animate | $log | $location | $rootElement |
| $cacheFactory | $filter | $templateCache | $rootScope |
| $compile | $http | $templateRequest | $sce |
| $controller | $httpBackend | $timeout | $sceDelegate |
| $document | $interpolate | $interval | $parse |
| $window |

### Cách sử dụng Services

Sử dụng Services bằng cách bơm truyền (inject) các đối tượng Service vào trong hàm khai báo của các thành phần trong Angular (controller, service, filter or directive). Bạn có thể tìm hiểu cách bơm truyền (inject) qua bài viết [dependency injection](http://acegik.net/blog/javascript/angularjs/tong-quan-ve-dependency-injection-trong-angularjs-p1.html).

<ins>Ví dụ:</ins>

```js
var myApp = angular.module('demoModule', []);

myApp.controller('demoController', ['$window', function(win){
    win.alert('This is the demo for using $window Service');
}]);
```
Hay

```js
var myApp = angular.module('demoModule', []);

myApp.controller('demoController', function($window){
    $window.alert('This is the demo for using $window Service');
});
```


### Cách tạo một Custom Service


#### Sử dụng method Provider để khai báo

```js
myApp.provider('createdByProvider', function() {
    this.$get = function() {
        return {
            myValue : "Using Provider method must be return an object inside of '$get' method",
            showText: function() {
                return "This text was created by Provider function";
            }
        }
    };
});
```

Tham số thứ nhất ‘createdByProvider’ là tên của Service cần tạo, tham số thứ hai là function(){…} định nghĩa **method this.$get** xử lý trả về kết quả là một đối tượng (object).


#### Sử dụng method Factory để khai báo

```js
myApp.factory('createdByFactory', function() {
    return {
        myValue : "Using Factory method must be return an object",
        showText: function() {
            return "This text was created by Factory function";
        }
    };
});
```

Tham số thứ nhất ‘createdByFactory’ là tên của Service cần tạo, tham số thứ hai là function(){…} xử lý trả về kết quả là một đối tượng (object).


#### Sử dụng method Service để khai báo

```js
myApp.service('createdByService', function() {
    this.myValue = "Using Service method will return an object";
    this.showText = function() {
        return "This text was created by Service function";
    };
});
```

Tham số thứ nhất ‘createdByService’ là tên của Service cần tạo, tham số thứ hai là function(){…} định nghĩa một class.



### Sự khác nhau giữa khai báo Service bằng các method Provider, Factory và Service

Cả 3 cách khai báo được trình bày ở phần trên đều có thể tạo được một Service. Nhưng sự khác nhau giữa 3 cách khai báo trên là gì? Chúng ta sẽ tìm hiểu thông qua source code của AngularJS v1.3.8.


#### Source code của method provider

```js
function provider(name, provider_) {
    assertNotHasOwnProperty(name, 'service');
    if (isFunction(provider_) || isArray(provider_)) {
        provider_ = providerInjector.instantiate(provider_);
    }
    if (!provider_.$get) {
        throw $injectorMinErr('pget', "Provider '{0}' must define $get factory method.", name);
    }
    return providerCache[name + providerSuffix] = provider_;
}
```

Tham số “name” là một string, tham số “provider_” là một trong 3 thành phần sau:

    - Function: Phải trả về một đối tượng có method “$get”.
    - Array: Được coi như sử dụng inline Annotation, và phải trả về một đối tượng với method “$get”.
    - Object: Nếu là object thì phải có method “$get”.

Như vậy, tham số thứ hai “provider_” cho dù là gì thì cũng phải có chứa khai báo method $get.


#### Source code của method factory

```js
function factory(name, factoryFn, enforce) {
    return provider(name, {
        $get: enforce !== false ? 
              enforceReturnValue(name, factoryFn) : factoryFn
    });
}
```

Method factory thực thi method provider để trả lại kết quả. Tham số thứ hai được truyền vào method provider là một đối tượng có khai báo method $get, method $get được khai báo bằng tham số thứ hai của method factory. Tham số thứ ba sẽ check để gán giá trị cho method $get, mặc định sẽ là ‘!==false’ nếu không khai báo và thực thi ‘enforceReturnValue’ trả về kết quả, ngược lại sẽ gán ‘functionFn’ cho method $get.

Có thể nói method factory là bản rút gọn của method provider cho trường hợp tham số thứ hai là đối tượng (object) có chứa method $get.


#### Source code của method service

```js
function service(name, constructor) {
    return factory(name, ['$injector', function($injector) {
        return $injector.instantiate(constructor);
    }]);
}
```

Method service thực thi method factory để trả lại kết quả. Tham số thứ hai được truyền vào method factory là một array (kiểu khai báo sử dụng Inline Annotation), sử dụng service $injector để khởi tạo đối tượng bằng method instantiate, tham số được truyền vào method instantiate là tham số thứ hai của method service. 

Các bạn hãy xem ví dụ sau đây để thấy sự khác biệt giữa 3 đối tượng được tạo từ method Provider, Factory và Service.

<ins>Ví dụ:</ins>

<iframe src="http://embed.plnkr.co/KUElg1NmWi1KDerqK4Gj/preview" style="border:1px #FFFFFF none;" name="myiFrame" scrolling="no" frameborder="1" marginheight="0px" marginwidth="0px" height="230px" width="100%"></iframe>

Theo ví dụ trên, cả 3 cách khai báo đều dùng chung một hàm khai báo “MyFn”, nhưng khi được gọi sử dụng trong Directive thì cho ra kết quả khác nhau. 

Đối tượng được khai báo bởi:

    - Method Factory: Trong mỗi lần được gọi, function MyFn được thực thi trả về kết quả.
    - Method Provider: Trong mỗi lần được gọi, function $get bên trong khai báo MyFn được thực thi trả về kết quả.
    - Method Service: Tạo ra một đối tượng singleton dựa vào khai báo MyFn.

Như vậy theo định nghĩa Angular Service ở phần đầu [Angular Services là gì?](#whatissevice), thì một Service phải là một đối tượng singleton. Vì vậy, nếu sử dụng một trong ba method Provider, Factory hay Service để khai báo một Service thì phải chú ý đến tham số thứ hai (đã đề cập ở phần [Cách tạo một Custom Service](#createService)).

    - Method Factory: Khai báo function phải trả về một đối tượng.
    - Method Provider: Khai báo function chứa method $get trả về một đối tượng.
    - Method Service: Khai báo function như định nghĩa một đối tượng.



### Tài liệu tham khảo
Bài viết giới thiệu về Angular Services có tham khảo các nguồn tài liệu sau:
    1. [https://docs.angularjs.org/guide/services](https://docs.angularjs.org/guide/services)
    2. [http://www.ng-newsletter.com/posts/beginner2expert-services.html](http://www.ng-newsletter.com/posts/beginner2expert-services.html)
    3. [http://slides.wesalvaro.com/20121113/#/2/3](http://slides.wesalvaro.com/20121113/#/2/3)
    4. [Tìm hiểu về Dependency Injection trong AngularJS](http://acegik.net/blog/javascript/angularjs/tong-quan-ve-dependency-injection-trong-angularjs-p1.html)
