---
title: AngularJS中的radio和checkbox
date: 2017-03-28 11:31:26
tags:
    - Angular
    - 前端
categories:
    - problem solving
---

# 场景
<img src="/blogImg/angular_radio.png" alt="angular_radio" width="400px;">
我们经常会遇到这样的场景， 有这样的图片数组对象，dom遍历出来，而用户又需要选择一个主图，因此遍历出来的单选按钮是一个组合，只能有一个被选中。于是乎，我们的问题就来了。
<!--more-->

---

# radio
## 我所以为的
```html
<html ng-app="myApp">
<body ng-controller="TestController">
    <div ng-repeat="img in imgList">
      {{img.txt}}
      <input type="radio" value="1" ng-model="img.is_default" name="img"/>
    </div>
    <button ng-click="submit()">提交</button>
</body>
</html>
```
```js
var myAppModule = angular.module('myApp', []);
myAppModule.controller('TestController', function($scope){
  $scope.imgList = [
      {txt:'111', is_default:0},
      {txt:'222', is_default:0},
      {txt:'333', is_default:0}
    ];
  
  $scope.submit= function(){  
    console.log($scope.imgList);
  }
  
});
```
demo效果如图：
<img src="/blogImg/angular_radio2.png" alt="angular_radio" width="100px;">
刚开始自己没有认识到radio的傲娇之处，balabala一顿狂写，然后发现了神奇的地方。单选的目的是达到了，但是用户只要选择过该选项，该选项的值将永远是1。
比如：我选过111，再去选择222，按了提交按钮。此时，打印出来的数组对象是这样的：
```
[
  {txt:'111', is_default:'1'},
  {txt:'222', is_default:'1'},
  {txt:'333', is_default:0}
];
```

## 解决方案
因此，单选图这个model就必须提出来做一个单独的设定了，我做了如下更改：
```html
<html ng-app="myApp">
<body ng-controller="TestController">
    <div ng-repeat="img in imgList">
      {{img.txt}}
      <input type="radio"  value="{{$index}}" ng-model="defaultImg.index" name="img"/>
    </div>
    <button ng-click="submit()">提交</button>
</body>
</html>
```
```js
var myAppModule = angular.module('myApp', []);
myAppModule.controller('TestController', function($scope){
  $scope.defaultImg = {
    index: -1
  }
  $scope.imgList = [
      {txt:'111', is_default:false},
      {txt:'222', is_default:false},
      {txt:'333', is_default:false}
    ];
  
  $scope.submit= function(){
    if($scope.defaultImg.index === -1){
      return; //没有选择balbala
    }
    var result = angular.copy($scope.imgList);
    result[index].is_defalut = true;

    console.log(result);
  }
});
````
最后得到的result数组完美达到了我们想要的效果。

## why
可是为什么呢？为什么radio在name一样的情况之下，绑不通的model，并没有按照我们所理解的去设值呢。
首先，在对radio不设置value的情况之下，ng-model取到的是undefined，而并不是我们以为的boolean。
[官网](https://docs.angularjs.org/api/ng/input/input%5Bradio%5D)上对radio的value解释是这样说的：
> The value to which the ngModel expression should be set when selected.

也就是说，当radio被选择的时候，会相对应影响绑定在其上的ngModel。在上面最开始的情况代码中，每个radio绑定不通的ngModel,只有点击操作会影响该radio本身ngModel的值，并不会对其他radio造成影响。也就出现了，只要之前那种只要点击过，is_default都会变成1的情况。

---
# checkbox
那么相对应的，咱们也来讲讲复选框。
[官网解释](https://docs.angularjs.org/api/ng/input/input%5Bcheckbox%5D)：
```html 
<input type="checkbox"
       ng-model="string"
       [name="string"]
       [ng-true-value="expression"]
       [ng-false-value="expression"]
       [ng-change="string"]>
```
与单选框不同的是，复选框默认的值就是true或false。
Angular官网还给了一种配置是否选中设置不同值的用法。
```html
<input type="checkbox"  ng-model="img.is_default" ng-true-value="1" ng-false-value="0"/>
```
选中复选框的时候，is_default的值便是1， 弃选的值是0。
当然这里需要理解一下什么叫弃选。
如果是用户从来没点击过，那么checkbox的ngModel便是绑定对象的初始值。而不是ng-false-value设置的值。
> [ng-true-value="expression"]

ng-true-value会将对应数字和boolean进行转换。
```js
ng-true-value="true" === true
ng-true-value="1" === 1
//定义字符串的方式
ng-true-value="'aaa'" === 'aaa'
```

# 默认选中
用radio和checkbox时的时候，经常会有默认选中这样的必要。那么分别是怎么处理的呢。
* radio
```html
<input type="radio"  ng-model="img.is_default" value="{{$index}}"/>
当默认img.is_default与value的值严格相等时，便会默认选中。
严格相等 : ===
```
* checkbox
```html
<input type="checkbox"  ng-model="img.is_default" ng-true-value="1" ng-false-value="0"/>
当默认img.is_default与ng-true-value的值严格相等时，便会默认选中。
```
* ng-checked
最后还有一个终极大法，就是ng-checked。具体就不赘述了，请看[官网](https://docs.angularjs.org/api/ng/directive/ngChecked)。

