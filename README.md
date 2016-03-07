**Important**
- `master` branch contains the bleeding edge development code.
- check `branches` or `tags` for the latest stable release or specific versions.

# Paloma

Page-specific javascript for Rails done right.

## Advantages
* Choose what specific javascript code to run per page.
* Easily make ruby variables available on your javascript files.
* Can be written using vanilla javascript, coffeescript, and anything that compiles to js.
* No external library dependency.
* Easy to understand (*because it is patterned after Rails' controller module*).

## Quick Example

Paloma controller.

```javascript
Paloma.controller('Users', {
  new: function(){
    // Executes when Rails User#new is executed.
    alert('Hello Sexy User!');
  }
});
```
 
The Rails controller `app/controllers/users_controller.rb`:

```ruby
def UsersController < ApplicationController
    def new
      # nothing special
      @user = User.new
    end
end
```

That's it! Simply Sexy!

## Minimum Requirements
* Rails 3.1 or higher


## Install

1. Install gem.
  - Without bundler: `sudo gem install paloma`.
  - With bundler, add this to your Gemfile: `gem 'paloma'`

1. Require `paloma` in your `application.js`: `//= require paloma`

1. In your layouts insert Paloma's hook.

   `application.html.erb`
   ```html
   <html>
      <head>
      </head>
      
      <body>
         <%= yield %>
         <%= insert_paloma_hook %>
      </body>
   </html>
   ```

1. Start Paloma. *Most of the time this will be inside `document.ready`*.
  ```js
  $(document).ready(function(){
    // Execute Paloma's callback.
    Paloma.start();
  });
  ```

## Controllers

Controllers are classes that handle requests made by Rails Controllers. Each Rails Controller's action will be mapped to a specific Paloma Controller's action.


### Creating a Controller

A Controller constructor is created or accessed (if it already exists), using `Paloma.controller()` method.

```javascript
var ArticlesController = Paloma.controller('Articles');
``` 

It will return the constructor function of your controller.

Note: Using `Paloma.controller` method, you can access the same controller constructor across different files.


### Handling Actions

Every time a request to Paloma is made (A Rails Controller action is executed), an instance of a Paloma controller is created and the method responsible for the request will be invoked.
 
 Add actions directly to the constructor's prototype.
 
```javascript
var Articles = Paloma.controller('Articles');

Articles.prototype.edit = function(){
  // Handle edit article
};
```

Or pass the prototype value directly as the 2nd argument of the `controller` method.

```js
Paloma.controller('Articles', {
  edit: function(){
    // Handle edit article
  }
});
```

### Namespace

Namespaced controller should follow this format: `namespace/controller`.

Rails controller:
```ruby
class Admin::UsersController < ApplicationController
  def new
    @user = User.new
  end
end
```

Paloma controller:
```js
Paloma.controller('Admin/Users', {
  new: function(){
    // Handle new admin user
  }
});
```

### Controller Inheritance

It is also possible to create a controller that is a subclass of an existing controller, using the following syntax: 
`Controller < ParentController`

```js
Paloma.controller('Application', {
  index: function(){
    alert('Application: Index');
  },
  
  new: function(){
    alert('Application: New');
  }
});

Paloma.controller('Users < Application', {
  // Override Application's new action
  new: function(){
    alert('Users: New');
  }
});
```

## Advanced Usage

You can manipulate what controller/action should Paloma execute by calling `js` method **before** rendering.

1. Changing controller

   ```ruby
   class UsersController < ApplicationController
      def new
         @user = User.new
         js 'Accounts' # will use Accounts controller instead of Users controller
      end
   end
   ```

2. Changing action

   You can use the symbol syntax:
   ```ruby
   def new
      @user = User.new
      js :register # will execute register method instead of new
   end
   ```
   
   Or the string syntax:
   ```ruby
   def new
      @user = User.new
      js '#register'
   end
   ```

3. Changing controller and action.

   ```ruby
   def new
     @user = User.new
     js 'Accounts#register' # will execute Accounts#register instead of Users#new
   end
   ```

4. Changing controller with namespace.

   Paloma supports namespaces using '/' as delimiter.

   ```ruby
   def new
      @user = User.new
      js `Admin/Accounts` # will use Admin/Accounts controller instead of Users controller
   end
   ```
   
   ```ruby
   def new
      @user = User.new
      js 'Admin/Accounts#register' # will execute Admin/Accounts#register instead of Users#new
   end
   ```
   

## Passing Parameters

You can pass parameters to your Paloma Controller in two ways.

1. Passing a hash. (*parameters only*)
 
  ```ruby
  def show
    user = User.find params[:id]
       
    js :id => user.id
  end
  ```
   
2. Passing path and a hash.
  
  ```ruby
  def show
    user = User.find params[:id]
    
    js 'Admin/Users', :id => user.id
  end
  ```

Using both ways, you can access the passed params using the `params` property of your Paloma controller.

```javascript
Paloma.controller('Users', {
  show: function(){
    alert("User id: " + this.params.id);
  }
});
```


## Preventing Paloma Execution

If you want to Paloma not to execute in a specific Rails Controller action you need to pass `false` as the Paloma parameter.

```ruby
def edit
  @user = User.find params[:id]
  js false
end
```


## Controller-wide setup

You can call `js` outside Rails controller actions for global or controller-wide settings.


**Example:**

```ruby
class UsersController < ApplicationController
   js 'Accounts' # use Accounts controller instead of Users for all actions.


   def new
      @user = User.new
   end
   
   def show
      @user = User.find params[:id]
   end
end
```

Like `before_filter` you can also pass `only` and `except` options.


```ruby
class UsersController < ApplicationController

   js 'Admin/Accounts', :except => :destroy # Use Admin/Accounts except for destroy method

end
```


**IMPORTANT NOTE:**
If you are going to pass parameters for Controller-wide settings, put them inside a `:params` hash.

```ruby
class UsersController < ApplicationController
  js 'Accounts', :params => {:x => 1, :y => 2, :z => 3}, :only => :show
end
```

### Overriding Controller-wide setup

If you want to override the controller-wide setup, just call `js` again inside a controller action. From there you can override the controller/action or pass additional parameters.

```ruby
class UsersController < ApplicationController

   js 'Accounts', :params => {:x => 1}
   
   
   def new
      @user = User.new
      js :register, :y => 2 # will execute Accounts#register with params {:x => 1, :y => 2}
   end
end
```

## Hook

`insert_paloma_hook` is a helper method that you can use in your views to insert Paloma's HTML hook.
Inside this HTML hook is where the magic happens. This is the reason why Paloma can magically know what Javascript controller/action to execute. To further understand how Paloma works, you can inspect the HTML hook, by checking the generated HTML (*inspect element*) and locate the `div` element that has the class `js-paloma-hook`.

Ideally, you just need to call `insert_paloma_hook` in your layouts, since the layout will always be included in every rendered view. But if you are rendering a view without a layout, make sure to call `insert_paloma_hook` in that view.


## AJAX

1. Make sure that the AJAX response contains the html hook. (use `insert_paloma_hook`)
2. Start Paloma on complete/success.

   ```js
   $.get('http://example.com', function(response){
      $('#result').html(response);
      Paloma.start();
   });
   ```

## Turbolinks Support

### Execute Paloma when user hits `Back` or `Forward` button.

Paloma executes page-specific javascript by adding a `<script>` tag to the response body. Turbolinks, by default, executes any inline javascript in the response body when you visit a page, so the `<script>` tag appended by Paloma will automatically be executed. However, when Turbolinks restores a page from cache (*this happens when a user hits `Back` or `Forward` button in his browser*) any **inline javascript will not be executed** anymore. This is the intentional behavior of Turbolinks, and it is not a bug. If you want to execute Paloma again when Turbolinks restores a page, do something like this:

```js
$(document).on('page:restore', function(){
  Paloma.start();
});
```

## Gotchas

* Make sure that the rendered view has the paloma hook (*use `insert_paloma_hook`*) for Paloma to execute. 


## Where to put code?

Again, Paloma is now flexible and doesn't force developers to follow specific directory structure.
You have the freedom to create controllers anywhere in your application.

Personally, I prefer having a javascript file for each controller.


## Contribute

1. Fork.
2. Do awesome things.
3. Submit Pull-Request to `master` branch.
4. Add short summary of changes on your PR.
