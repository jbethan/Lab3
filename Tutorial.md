# Avatar Tutorial

We will build a contrived user registration that will take

1. new user's name and

2. a link to a picture they want to use for an avatar

The added users will then display below the form.

This will help us

1. reinforce experience with Angular's powerful two-way data binding

2. Introduce us to a very powerful concept that Angular makes possible with its `directive` feature. Angular Directives allow us the ability to encapsulate UI style and behavior into reusable components. This is a critical aspect to scalable web applications.

We will begin by implementing the functionality with an Angular template and controller first, then refactor to include an `Avatar` directive.
Start with this html in "avatar.html"
```html
<!DOCTYPE html>
<html>
<head>
  <!-- include our dependency on angular -->
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.0/angular.min.js"></script>
<script src="avatar.js"></script>
  <meta charset="utf-8">
  <title>User Avatar Template</title>
</head>
<body>
  <!--
    1. ng-app='app' tells angular that everything within this div should function within
    the scope of a declared angular module with the name of app.
    2. ng-controller='mainCtrl' tells angular that the scope of this div should be handled
    by a controller by the name of 'mainCtrl'. In a bigger application you will likely have
    multiple controllers within one ng-app
  -->
  <div ng-app='app' ng-controller='mainCtrl'> <!-- [1],[2] -->
    <h1>Create new user</h1>
    
    <!--
     1. Giving a form a name in Angular is kind of like creationg a javascript object with that name:
     e.g. var userForm = {}
     2. This is the function with the given parameter we want it to be called with when the form is submitted
     'addNew' should be found on the scope of mainCtrl.
     3. .name & .url of userForm is like adding properties to our userForm object. Angular will automatically
     bind values to these.
     e.g. userForm = {
      name: '',
      url: ''
     }
    -->
    <form name='userForm' ng-submit='addNew(userForm)'> <!-- [1], [2] -->
      
      <input placeholder='Name' ng-model='userForm.name'/> <!-- [3] -->
      <input placeholder='Image Url' ng-model='userForm.url'/>
      
      <!-- 
      setting button to type submit will trigger the ng-submit of the 
      form that button is a child of.
      -->
      <button type='submit'>Add New</button>
    </form>
  </div>
</body>
</html>
```
And this controller in "avatar.js"
```js
angular.module('app', [])
  .controller('mainCtrl', mainCtrl);

function mainCtrl ($scope) {
  
  /**
   * Let's just make sure something happens when the user submits the form
   * by binding the declared 'addNew' function to the scope. You can see we
   * are expecting a user object as a parameter. This is 'userForm'
   */
  $scope.addNew = function (user) {
    alert(user.name + ' ' + user.url);
  };
}
```
Cool. Now we will:

1. bind an array of users to the `$scope` in mainCtrl

2. add a user to this array whenever `addNew` is called & clear the form

3. use `ng-repeat` to output an `avatar` for each user in our list of users

4. add some minimal style 

First the avatar.js
```js
function mainCtrl ($scope) {
  
  // tada! a users array will now be bound to and made available to our html template
  $scope.users = [];
  
  /**
   * 1. We push an new `user` object to our users list with a name
   * and avatarUrl property
   * 2. For our purposes, a quick and dirty method for clearing the form will due.
   * Just note there is a more appropriate method when form validation is involved.
   */
  $scope.addNew = function (user) {
    $scope.users.push({ 
      name: user.name,
      avatarUrl: user.url
    }); /* [1] */
    
    user.name = ''; /* [2] */
    user.url = ''; /* [2] */
  };
}
```
And the avatar.html
```html
<!DOCTYPE html>
<html>
<head>
  <!-- include our dependency on angular -->
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.0/angular.min.js"></script>
<script src="avatar.js"></script>
<link rel="stylesheet" type="text/css" href="avatar.css">
  <meta charset="utf-8">
  <title>User Avatar Template</title>
</head>
<body>
  <div ng-app='app' ng-controller='mainCtrl'>
    <h1>Create new user</h1>

    <form name='userForm' ng-submit='addNew(userForm)'>
      <input placeholder='Name' ng-model='userForm.name'/>
      <input placeholder='Image Url' ng-model='userForm.url'/>
      <button type='submit'>Add New</button>
    </form>
    
    <ul class='User-list'>
      <!--
        ng-repeat loops through the users array bound to the scope of mainCtrl and gives
        us access to the user object in each index so we can reference the user properties
      -->
      <li ng-repeat='user in users'>
        <div class='Avatar'>
          <img ng-src='{{user.avatarUrl}}' />
          <h4>{{user.name}}</h4>
        </div>
      </li>
    </ul>
  </div>
</body>
</html>
```
and avatar.css
```css
.User-list {
  list-style: none;
}

.Avatar {
  text-align: center;  
}

.Avatar img {
  width: 50px;
  height: 50px;
  border-radius: 50%;
}

.Avatar h4 {
  text-transform: capitalize;
  color: red;
}
```
Great! we should have have a simple user form that allows us to add styled user avatars

Let's take a moment to consider here some additional features we may like with our user avatar: 
1. what if a user doesn't want to upload an avatar? We don't want to show a broken image. We may want to make sure our avatars still look ok by using a default avatar instead. 
2. what about different avatar sizes to use on the main profile page, the navigation, or a user registry? How do we tell our avatar to render differently? 
3. What if later we added usernames and also wanted to display that within our avatar

Can you see how this may get messy quickly trying to handle this within our templates and mainCtrl? Also, doesn't it just feel wrong that these general purpose things should be worried about avatar stuff?

It would be nice hide away all of the style, behavior, and implementation details into a neat package labeled avatar component and let that worry about all of this.

We will now refactor our code to use an Angular directive instead

First refactor the html
```html
<!--
 We've clearly gone mad and have replaced the div, img, and h4 tags with what
 appears to be a mythical `avatar` tag.
 Note we have a property of user, passing a value of user. This will be the user object
 from the ng-repeat
-->
<ul class='User-list'>
  <li ng-repeat='user in users'>
    <avatar user='user' />
  </li>
</ul>
```
And then the js controller
```js
/**
 * 1. We have added a directive with the name 'avatar' and handler of
 * avatarDirective to our angular app module
 */
angular.module('app', [])
  .controller('mainCtrl', mainCtrl)
  .directive('avatar', avatarDirective);

function mainCtrl ($scope) {

  $scope.users = [];

  $scope.addNew = function (user) {
    $scope.users.push({ 
      name: user.name,
      avatarUrl: user.url
    }); /* [1] */
    
    user.name = ''; /* [2] */
    user.url = '';
  };
}

/**
 * 1. this defines the api of our avatar directive. This means we are
 * expecting a user property whose value should be interpreted as an object.
 * 2. This simply means we want this directive to be used as an element.
 * 3. You can see here we've moved the html that was in our template before
 * and give it as the template for this directive. This means wherever we use
 * <avatar /> this html will also be placed there.
 * 4. Here we are implementing the feature where if there is no user avatar url,
 * we go ahead and give it a default
 */
function avatarDirective () {
  return {
    scope: {
      user: '=' /* [1] */
    },
    restrict: 'E', /* [2] */
    replace: 'true',
    template: (
      '<div class="Avatar">' +
        '<img ng-src="{{user.avatarUrl}}" />' +
        '<h4>{{user.name}}</h4>' +
      '</div>'
    ), /* [3] */
    link: link
  };
  
  function link (scope) { /* [4] */
    if (!scope.user.avatarUrl) {
      scope.user.avatarUrl = 'https://www.drupal.org/files/issues/default-avatar.png';
    }
  }

}
```
