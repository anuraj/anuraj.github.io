---
layout: post
title: "Angular JS CRUD operations with ASP.NET5 - Part 3"
subtitle: "Angular JS CRUD operations with ASP.NET5 - Angular In Play"
date: 2015-11-24 12:00:00
categories: 
   - Angular JS
   - ASP.NET5
   - Unit Testing
   - Code coverage
   - Javascript
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
As I mentioned in the last post, this post is about client side implementation. Hope you have a basic understanding about Angular JS. If not, please have look into the [Angular JS tutorial](https://docs.angularjs.org/tutorial) page. Angular JS comes with two Ajax implementations to communicate to server. One $http and $resource. In this post I am using $http service.

Here is my Angular module
{% highlight Javascript %}
var todoApp = angular.module("todoApp", []);
{% endhighlight %}

Here is my controller, which is interacting with the ASP.NET 5 REST Service.

{% highlight Javascript %}
todoApp.controller("todoController", function ($scope, $http) {
	var formData = {
		Description: "default",
		IsCompleted: false
	};

	$scope.loadTodos = function () {
		$http.get("/Home/Todos").then(function (response) {
			$scope.ToDos = response.data;
		});
	};
	$scope.loadTodos();
	$scope.markAsCompleted = function (todo) {
		$http.post("/Home/MarkAsCompleted", todo).then(function (response) {
			console.log('Updated');
		});
	};

	$scope.CreateTodo = function () {
		$http.post("/Home/CreateTodo", $scope.form).then(function (response) {
			$scope.ToDos.push(response.data);
			$scope.form.Description = "";
		});
	};

	$scope.deleteTodo = function (todo) {
		$http.post("/Home/DeleteTodo", todo).then(function (response) {
			for (var i = 0, iLen = $scope.ToDos.length; i < iLen; i++) {
				if ($scope.ToDos[i].Id == response.data.Id) {
					$scope.ToDos.splice(i, 1);
					break;
				}
			}
		});
	};
});
{% endhighlight %}

In the first line, Angular JS will inject the $scope and $http services to the controller, scope is the glue between application controller and the view. And the http service will help the client app to interact with Service. And in the HTML you need to assign the App and controller.

In this code I am disabling the submit button, if the form is not valid. And I am using ng-submit, which will invoke the CreateToDo method when the form is getting submitted.

{% highlight HTML %}
<div ng-app="todoApp" ng-controller="todoController">
    <form name="createToDoForm" role="form" ng-submit="CreateTodo()" novalidate>  
    <div class="row">
      <div class="col-xs-12">
        <div class="input-group input-group-lg">
            <input name="Description" type="text" class="form-control" ng-model="form.Description" required />
            </span>
            <div class="input-group-btn">
            <button type="submit" ng-disabled="createToDoForm.$invalid" class="btn">Add</button>
          </div><!-- /btn-group -->
        </div><!-- /input-group -->
      </div><!-- /.col-xs-12 -->
    </div><!-- /.row -->
</form>
<hr/>
<form>
    <table class="table table-hover table-striped">
        <tr ng-repeat="todo in ToDos">
        <td class="col-md-1">
            <input type="checkbox" ng-checked="todo.IsCompleted" ng-click="markAsCompleted(todo)" />
        </td>
        <td class="col-md-8">
            <h4>{{ todo.Description }}</h4>
        </td>
        <td class="col-md-1">
            <i class="fa fa-trash-o fa-3x" ng-click="deleteTodo(todo)"></i>
        </td>
        </tr>
    </table>
</div>
</form>
{% endhighlight %}

And here is the app running on my system.

![Angular Todo List app running]({{ site.url }}/assets/images/2015/11/todolistapp.png)

In the next post I will try to cover unit testing in Angular JS.