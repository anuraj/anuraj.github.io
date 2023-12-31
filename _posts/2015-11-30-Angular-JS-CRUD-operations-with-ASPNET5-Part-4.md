---
layout: post
title: "Angular JS CRUD operations with ASP.NET5 - Part 4"
subtitle: "Angular JS CRUD operations with ASP.NET5 - Unit testing"
date: 2015-11-30 12:00:00
categories: 
   - Angular JS
   - ASP.NET5
   - Unit Testing
   - Code coverage
   - Javascript
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about Angular JS unit testing. This post is using [Jasmine](http://jasmine.github.io/); Jasmine is a behavior-driven development framework for testing JavaScript code. For testability I have modified the controller code little bit. Here is the controller.

{% highlight Javascript %}
todoApp.controller("todoController",["$scope", "$http",function ($scope, $http) {
	
	$scope.Todo = {
		Description: "",
		IsCompleted: false
	};
	
	$scope.ToDos = [];
	$scope.loadTodos = function () {
		$http.get("/Home/Todos").then(function (response) {
			$scope.ToDos = response.data;
		});
	};
	
	$scope.markAsCompleted = function (todo) {
		$http.post("/Home/MarkAsCompleted", todo).then(function (response) {
			$scope.ToDos.push(response.data);
			$scope.Todo.Description = "";
		});
	};

	$scope.CreateTodo = function () {
		$http.post("/Home/CreateTodo", $scope.Todo).then(function (response) {
			$scope.ToDos.push(response.data);
			$scope.Todo.Description = "";
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
}]);
{% endhighlight %}

Major changes are like instead of using Form variable, I have created a scope variable. Next you need to create test runner. You can download the Jasmine files and reference it in a HTML file. You can use this HTML file to view the unit test results. I have included Jasmine and Angular Mocks in bower.json and install it via bower install command. Here is my bower.json file

{% highlight Javascript %}
{
  "name": "todolist",
  "private": true,
  "dependencies": {
    "bootstrap": "3.0.0",
    "angular": "*",
    "font-awesome": "*",
    "jasmine": "*",
    "angular-mocks": "*"
  }
}
{% endhighlight %}

And here is the Test runner file, with references of the App, Controller and other references. Make sure you're not changing the order, otherwise Jasmine won't work.

{% highlight HTML %}
@{
    Layout = null;
}
<html>
    <head>
        <title>Test Runner</title>
        <link rel="stylesheet" type="text/css" href="~/lib/jasmine/lib/jasmine-core/jasmine.css">
        <script type="text/javascript" src="~/lib/jasmine/lib/jasmine-core/jasmine.js"></script>
        <script type="text/javascript" src="~/lib/jasmine/lib/jasmine-core/jasmine-html.js"></script>
        <script type="text/javascript" src="~/lib/jasmine/lib/jasmine-core/boot.js"></script>
        <script src="~/lib/angular/angular.min.js"></script>
        <script type="text/javascript" src="~/lib/angular-mocks/angular-mocks.js"></script>
        
        <script src="~/js/todoApp.js"></script>
        <script src="~/js/todoController.js"></script>
    </head>
    <body>
    </body>
</html>
{% endhighlight %}

You can write unit tests after the controller references. You can find unit tests with *describe* and *it* global functions. Here is the first sample unit test.

{% highlight HTML %}
describe("Hello World", function(){
    it("First Spec", function(){
        expect(10+20).toBe(30);
    }); 
});
{% endhighlight %}

Here is the screenshot of hello world unit test.

![Screenshot of hello world unit test]({{ site.url }}/assets/images/2015/11/firstSpec1.png)

For mocking http service angular comes with httpBackend service. And here is the unit tests for CreateToDo and loadTodos functions.

{% highlight Javascript %}
describe("ToDo Controller - Tests", function(){
    it("loadTodos function should returns the list of todos", function(){
        
        var rootScope, controller,httpBackend, scope;
        angular.mock.module("todoApp");
        
        angular.mock.inject(function($rootScope, $controller,$httpBackend){
            rootScope = $rootScope;
            scope = rootScope.$new();
            controller = $controller("todoController",{ '$scope' : scope });
            httpBackend = $httpBackend;
        });
        
        httpBackend.whenGET("/Home/Todos").respond([{
            "Id":"1",
            "Description":"This a sample todo",
            "IsCompleted":"false"
        }]);
        
        httpBackend.expectGET("/Home/Todos");
        scope.loadTodos();
                            
        httpBackend.flush();
        httpBackend.verifyNoOutstandingExpectation();
        httpBackend.verifyNoOutstandingRequest();
        
        expect(scope.ToDos != null).toBe(true);
        expect(scope.ToDos.length).toBe(1);
        
    }); 
    
        it("CreateTodo function should create the todo", function(){
        var rootScope, controller,httpBackend, scope;
        angular.mock.module("todoApp");
        
        angular.mock.inject(function($rootScope, $controller,$httpBackend){
            rootScope = $rootScope;
            scope = rootScope.$new();
            controller = $controller("todoController",{ '$scope' : scope });
            httpBackend = $httpBackend;
        });
        
        var todo = {
            "Id":"1",
            "Description":"This a sample todo"
        };
        
        scope.Todo = todo;
        httpBackend.expectPOST("/Home/CreateTodo", todo).respond(200, todo);
        
        scope.CreateTodo(todo);                   
                            
        httpBackend.flush();
        
        httpBackend.verifyNoOutstandingExpectation();
        httpBackend.verifyNoOutstandingRequest();
        
        expect(scope.ToDos.length).toBe(1);
    }); 
});
{% endhighlight %}

Here is the screenshot 

![Screenshot of todo list unit tests]({{ site.url }}/assets/images/2015/11/todolistapptests.png)

If you look into the code, we are duplicating lot of code. We can refactor using *beforeEach* function. Here is the refactored version of the unit test code.

{% highlight Javascript %}
describe("ToDo Controller - Tests", function(){
    var rootScope, controller,httpBackend, scope;
    beforeEach(function(){ 
        angular.mock.module("todoApp");
        angular.mock.inject(function($rootScope, $controller,$httpBackend){
            rootScope = $rootScope;
            scope = rootScope.$new();
            controller = $controller("todoController",{ '$scope' : scope });
            httpBackend = $httpBackend;
        });
    });
    
    afterEach(function(){
        httpBackend.verifyNoOutstandingExpectation();
        httpBackend.verifyNoOutstandingRequest();
    });
    it("loadTodos function should returns the list of todos", function(){
        httpBackend.whenGET("/Home/Todos").respond([{
            "Id":"1",
            "Description":"This a sample todo",
            "IsCompleted":"false"
        }]);
        
        httpBackend.expectGET("/Home/Todos");
        scope.loadTodos();
                            
        httpBackend.flush();
        
        expect(scope.ToDos != null).toBe(true);
        expect(scope.ToDos.length).toBe(1);
        
    }); 
    
    it("CreateTodo function should create the todo", function(){
        var todo = {
            "Id":"1",
            "Description":"This a sample todo"
        };
        
        scope.Todo = todo;
        httpBackend.expectPOST("/Home/CreateTodo", todo).respond(200, todo);
        
        scope.CreateTodo(todo);                   
                            
        httpBackend.flush();
        expect(scope.ToDos.length).toBe(1);
    }); 
});
{% endhighlight %}

In the next post I will cover code coverage using blanket.js. Happy Coding.