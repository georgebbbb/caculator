前端文档
======================================
install
------------------------------------------
      npm install -g bower
      npm install -g gulp
      bower install
      npm   install
      gulp



1.包的分类
-------------------------------------
###1.1 最外层文件夹
app ：所有源代码
bower_components 前端依赖库
node_modules     构建依赖库
gulpfile.js      构建工具
package.json     npm配置文件
###1.2 app文件夹(代码组织)
Structure your app such that you can Locate your code quickly, Identify the code at a glance, keep the Flattest structure you can, and Try to stay DRY. The structure should follow these 4 basic guidelines.
这个单页应用的入口html为index.html
每个文件夹都是一个模块
其中除fxt－common和fxt－component外
其他模块全部按业务分并且都在fxt－index里面
比如说fxt－business为 业务设置 模块
业务设置模块下面
又有1.组织管理2.物业参数3.楼盘字典4.管理选项｀｀｀｀｀｀｀｀等6个模块所以会有
fxt-organization
fxt-parameter
fxt-option
fxt-dictionary
......几个模块
莫块下面如果没有模块了，那么这么模块就对应一个界面如果模块下面还有模块，那么会接着往下分模块比如说fxt-organization 下面会有组织结构和异动记录两个模块，所以会有
fxt-organization-manage
和
fxt-unusual-record
两个模块
但是fxt-organization-manage没有子模块了所以她就是一个页面
下面说一说两个特殊的模块
fxt-common所有公共组件都放到这里
啥叫公共组件呢，就是至少两个地方用到它的而且还单独包含业务逻辑，比如说楼盘字典那个组件好几个地方要用到，要尽可能的复用，这样的话维护的时候，只需要维护一份代码在比如写跟进
fxt-component一看名字就知道是放组件，为啥有了fxt－common还要有这个呢？这里放的是和业务逻辑无关的公用组件，我在这个系统可以用，改天我换新房系统了，这套组件照样可以拿来用这里面又详细的分了三个包
fxt-layout 这里放的是和布局有关的 比如包括每个panel，input的样式
fxt-custom 这里面放置一些订制组件，关于一些特效的，比如confirm
fxt-template 这里面放一些模版，比如说通过formly生成表单，每个表单需要的模版
###1.3 界面的组成
一个界面将会由很多组件组成，举例来说
fxt-organization-manage 会用这么几个：新增部门，修改部门(权限)，删除当前部门。。。。等等这么几个功能，那么每个功能模块独立的封装成各自的组件每个组件占一个文件夹，比如说
fxt-add-department
fxt-modify-department
那么除了这些组件外一班还会有
index.less
index.js
index.html
factory.js
那么
index.js 放controller的内容 关于controller 下面专题介绍
factory.js 放业务逻辑的内容，包括整个页面的数据， 专题介绍
index.html 这么就是这个页面的内容了
index.less  或者index.css 页面的样式
###1.4组件的组成
类比于页面的组成，组件的组成其实相似，不同的是每个组件的数据尽量来自于界面，why？ 如果每个组件都独自拿数据的话，各自为政，竟成会出现相同一个页面发起多次同样请求
其他的同界面一样，其实组件就可以把它当成一个小界面，拥有独立的mvvm。

2.最佳实践
------------------------------------------------------------------
###1.控制器 controller

不要在控制器里操作 DOM。通过指令完成。
通过控制器完成的业务界面命名控制器 (如：fxtOriganizationManage 组织结构)，为保持统一风格不要以controller结尾，而是应该起所在文件夹的名字相对应，比如说（如：fxtOriganizationManage所在文件夹为fxt-organization-manage，）。控制器采用驼峰命名法
因为每个界面就只需要一个controller（modal不算）,所以在每个界面文件夹下的index.js就表示该文件的controller

控制器的定义：

```javascript
angular.module('fxtBusiness').controller('fxtOrganizationManage', fxtOrganizationManage);


function fxtOrganizationManage($scope,organizationManage,notify) {
        }
```
对于控制器的依赖，我用构建工具可以直接生成。这样写的话，减少了嵌套，代码更整齐；


控制器就想j2ee中的controller层，所以不要把业务逻辑写在这里，而应该写在每个factory.js
 中，这样的话逻辑更清晰，controller中就不会出现一坨代码，很难维护
在需要格式化数据时将格式化逻辑封装成 过滤器 并将其声明为依赖：

```javascript
module.filter('myFormat', function () {
  return function () {
    //body...
  };
});

module.controller('MyCtrl', ['$scope', 'myFormatFilter', function ($scope, myFormatFilter) {
  //body...
}]);
```
强调，controller 越扁越好，这里只做数据的交汇，
controller里面无非就这么一下几个函数
del 删除
save 保存 包括新增的保存和修改的保存
add 新增

###2. 路由
没有特殊要求，页面所需要的数据要，在用 resolve 提前准备好，
对于非要在页面后再去数据的，那么如果子组件要用到该数据，要传给子组件一个promise，
why？
这样做就能规避大量的watch，提高性能
###3.directive
####1解耦合
写directive的关键就是解耦合,除了数据，必须依赖于上一层提供。这个是没办法的，但是其他的，不要依赖于上层，这样的话，如果出现的了bug，哪里出现bug改哪里就好了，也不会出现牵一发动全身。另外检验directive的最好办法就是，放到哪个页面，页面只提供必要数据，不加额外代码都不影响使用，
举例来说
新增部门，需要出发一个modal，而一般的做法是
在fxtOrganizationManage中加入如下代码
```javascript
angular.module('fxtBusiness').controller('fxtOrganizationManage', fxtOrganizationManage);


function fxtOrganizationManage($scope,$modal) {

    $scope.addDepartment=function(){

    var modalInstance = $modal.open({
                        templateUrl: 'fxt-business/fxt-organization/fxt-organization-manage/fxt-add-department/modal/index.html',
                        controller: 'fxtAddDepartmentModalController'});
                    }
}
```

但是，这样做的不好出显而易见，如果我在别的地方我要新增部门，我还要带着这些代码，而且，这样的我们也违反了controller的原则，controller也被污染了
比较好的做法是
定义一个fxt-add-department的directive


```javascript
angular.module('fxtBusiness').directive('fxtAddDepartment', ['$modal', function($modal) {
    return {
        restrict: 'A',
        link: function($scope, iElm, iAttrs, controller) {
            iElm.bind('click', function() {
                $scope.$apply(function() {
                    var modalInstance = $modal.open({
                        templateUrl: 'fxt-business/fxt-organization/fxt-organization-manage/fxt-add-department/modal/index.html',
                        controller: 'fxtAddDepartmentModalController',

                    });

                });

            });

        }
    };
}]);
```


然后html中 加入这个directive就ok了


```html
    <button type="button" fxt-add-department >新增部门</button>
```



而这个directive放到哪里都可以用，controller 也瘦身了

####2.代码组织
上面举例的directive的代码组织是不推荐的，推荐的方式是

```javascript
angular.module('fxtBusiness').directive('fxtAddDepartment', fxtAddDepartment);
 function fxtAddDepartment($modal) {
        return {
        restrict: 'A',
        link: link;
    };
};

function link($scope, iElm, iAttrs, controller) {
            iElm.bind('click', function() {
                $scope.$apply(function() {
                    var modalInstance = $modal.open({
                        templateUrl: 'fxt-business/fxt-organization/fxt-organization-manage/fxt-add-department/modal/index.html',
                        controller: 'fxtAddDepartmentModalController',
                    });
                });
            });
        }
```
why?
嵌套少了，代码更清晰！
####3.命名规则
为了体现咱的组件话，directive的命名应该和他所在的文件夹名相同，比如fxt-add-department，那么指令的命名为fxtAddDepartment，并且放在该文件夹的index.js文件夹下，后缀不要加directive,类比controller，controller也是和他所在的文件夹命名一样，也是放在index.js中，所以index.js中只会存在两种controller 和 directive,而且他俩不会同时存在，所以不需要加directive
####4.css和.html
对于这个directive对应的模版html的样式写在这里，不要写在全局，这样的话，css也能实现模块化。维护起来也更容易，css的.class命名要和directive标签一样,尽量减少.class的出现，尽量通过标签选择器，如果一定要出现的话，子class命名要以父clss为前缀，比如父class为fxt-orfanization-manage,子class为fxt-orfanization-manage－title， 这样避免冲突,不过能少用类选择器,尽量少用,可以用.fxt-orfanization-manage>h3代替。
####5.注意事项，
不要再link中出现赤裸裸jquery。








##tip

根据对象的编号排序的公共方法，在fxt－utiliy里面，他会根据对象的指定属性，先转成数字并给他从小到达排序

   COL_FIELD

   城市城区商圈级连复用fxt-cda


   新增通过factory练习新增的id



   每个input必须写name
   每个number类型数字必须初始化为‘’
