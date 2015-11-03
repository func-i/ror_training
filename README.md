# Rails App

This app is going to be a Project Management System that will use the server to render all templates.


## Create the Rails app

* $ rails new jonah_tracker
* $ cd jonah_tracker
* $ # Open in your editor
* Create your sqlite database
  * $ rake db:create
    * Where did it go?
    * db/developement.sqlite3
    * db/test.sqlite3
* Start your server
  * $ rails s
* open http://localhost:3000

## STOP HERE

We're going to talk about app structure.

## Create your routes

### Exercise

Rails has a routing layer that routes requests to "Controllers". (MVC for the win).

```
  config/routes.rb
```

Let's route our default route to the projects list.

If you want to keep all the comments in the router, feel free.  Otherwise replace the contents of the file with:

```
  Rails.application.routes.draw do
    root to: 'projects#index'
  end
```

* Reload your page
* View your routes
  * $ rake routes

**Oops!**  You need the controller.  Go ahead and create it.

### Questions

1. What is the routing file?
1. What is 'projects#index'?

## Create your controller

### Exercise

* $ rails generate controller projects index
  * *Generate the controller "projects" with the action "index"*
* app/controllers/projects_controller.rb
* app/views/projects/index.html.erb
* It also generates a **helper**, a **scss file**, a **coffeescript file**, and a **test**
  * *You don't have to use the generator, you can create these files yourself*
* It also added a route to **config/routes.rb**
* Reload your page

### Questions

1. What is an action?
1. Why does our page work now?

## Add some style

### Exercise

Let's quickly add boostrap to the project

* Stop your rails server (ctrl-c)
* Open your Gemfile (file in the root of your project)
* Add the line ```gem 'boostrap-sass'```
* $ bundle install
* Open ```app/assets/javascripts/application.js``` and add ```//= require bootstrap```
* mv app/assets/stylesheets/application.css app/assets/stylesheets/application.scss
* replace contents of ```applications.scss``` with:
```
  @import "bootstrap-sprockets";
  @import "bootstrap";
```
* $ rails s
* open ```app/views/projects/index.html.erb``` and add:
```
  <table class='table'>
    <thead>
      <tr>
        <th>Name</th>
      </tr>
    </thead>
  </table>
```

* Wrap yield in ```app/views/layouts/application.html.erb``` with a container:
```erb
  <div class='container'>
    <%= yield %>
  </div>
```

### Questions

Skip, we'll talk more about this later on

## Add some content

### Exercise

* In projects_controller.rb add:
```ruby
  def index
    @projects = [
      {name: "Larry"},
      {name: "Curly"},
      {name: "Moe"}
    ]
  end
```

* Add to ```app/views/projects/index.html.erb```
```erb
  <tbody>
    <% @projects.each do |project| %>
      <tr>
        <td><%= project[:name] %></td>
      </tr>
    <% end %>
  </tbody>
```

### Questions

1. What type of object is {name: "Larry"}
1. What type of variable is @projects?
1. What is ```<% %>```?
1. What is ```<%= %>```?
1. What is the difference between the two?

## STOP HERE

We're going to talk about ActiveRecord

## Data Objects we will create

### Exercise

* Project
* Task
* Developer

* $ rails g model Project
* View the "migration" ```db/migrate/XXXX_create_projects.rb```
* Add ```t.string :name``` to the "migration"
* $ rake db:migrate
* Change the projects controller
```ruby
  def index
    @projects = Project.all
  end
```
* Change the index.html.erb
```erb
  <% @projects.each do |project| %>
    <tr>
      <td><%= project.name %></td>
    </tr>
  <% end %>
```
* Load your console
  * bundle exec rails c (or maybe just rails c)
* Create a new Person
  * $> Project.create(name: "Functional Imperative")
* Reload your app

### Questions

1. What is a migration?
1. Why is it useful?
1. What happens if you run ```rake db:migrate``` a second time?
1. What does ```Project.all``` do?
1. What sql statement does it execute?
1. What does Project.create do?
1. What sql statement does it execute?
1. Why are we using ```project.name``` and not ```project[:name]``` now?

**Yay!**

## Viewing a record (routes and helpers)

### Exercise

* Add to index.html.erb
```erb
  <td>
    <%= link_to "View", project, class: 'btn btn-primary' %>
  </td>
```
* Add to your routes.rb
```ruby
  get 'projects/:id', to: 'projects#show', as: :project
```
* Add to your projects_controller.rb
```ruby
  def show
    @project = Project.find params[:id]
  end
```
* Create a app/views/projects/show.html.erb
  * What do you think goes in here???
* When done, go to your terminal, look at the server log.  
  * Notice the parameters
  * Notice the SQL that was executed
  * Notice the time spent rendering and querying

### Questions

* What is a link_to?
  * What tag does it create? (use web inspector in browser)
  * Why do you think we use it?
  * What is the second parameter ```project```
  * Does it work if you replace it with this:
    * ```<%= link_to "View", project_path(project), class: 'btn btn-primary' %>```
    * What is ```project_path```?
* View your routes
  * $ rake routes
  * Notice the "project" route.  
  * That's where ```project_path``` comes from.

## Creating new records

### Exercise

* Create ```app/views/projects/new.html.erb```
```erb
  In new
```
* Create the route
```get 'projects/new', to: 'projects#new', as: :new_project```
  * Visit the url /projects/new
  * What happens if you add it as the last route?
  * What happens if you add it above ```get '/projects/:id'```?
* Add the button to the projects index.html above the table
  ```erb
    <%= link_to "New Project", new_project_path, class: 'btn btn-success pull-right' %>
  ```
* Add the form
  * Replace the content of new.html.erb with
  ```erb
    <%= form_for @project do |f| %>
      <div class="form-group">
        <%= f.label :name %>
        <%= f.text_field :name, class: 'form-control' %>
      </div>
      <%= f.submit "Create", class: 'btn btn-success' %>
    <% end %>
  ```
* Add the following code to your ```projects_controller.rb```
```ruby
  def new
    @project = Project.new
  end
```
* Add the following to your ```projects_controller.rb```
```ruby
  def create
    @project = Project.new project_params
    if @project.save
      redirect_to @project
    else
      render :new
    end
  end

  protected 

  def project_params
    params.require(:project).permit(:name)
  end
```
* Submit your form, missing the route
* Add the route:
```ruby
  post 'projects', to: 'projects#create'
```

### Questions

1. Based on the problem you had with the ```projects/new``` route, how are the routes read?
1. Why didn't it find the new route?
1. Why do you think we don't need ```def new``` in our controller?
1. What is form_for
1. What is the action set to?
1. What are project_params (strong params)
1. Create another new record
  * Look at the rails server log
  * What is the authenticity_token?
  * Why do the params look like project: {name: ""}?
    * Inspect the form elements
1. In the controller, what does the if statement do in ```create```?
1. What happens if we don't save it?
  * How do you think it's possible it fails saving?
1. What does the render do?
1. Why ```:new``` and not ```:create```?

## Updating existing records

### Exercise

* Create ```app/views/projects/edit.html.erb```
```ruby
  <%= form_for @project do |f| %>
    <div class="form-group">
      <%= f.label :name %>
      <%= f.text_field :name, class: 'form-control' %>
    </div>
    <%= f.submit "Save", class: 'btn btn-success' %>
  <% end %>
```
* Add to your ```config/routes.rb```
```
  get 'projects/:id/edit', to: 'projects#edit', as: :edit_project
```
* Add to your ```app/views/projects/show.html.erb```
```
  <%= link_to "Edit", edit_project_path(@project), class: 'btn btn-warning' %>
```
* Submit the form
  * Oops, let's create the route and the controller action
* Add the following route
```ruby
  put 'projects/:id', to: 'projects#update'
  patch 'projects/:id', to: 'projects#update'
```
* Add the following to your ```projects_controller.rb```
```ruby
  def update
    @project = Project.find params[:id]
    if @project.update project_params
      redirect_to @project
    else
      render :edit
    end
  end
```

### Questions

1. What is the action for the form?
1. What is the method of the form?
1. Look at the rails server log, what is the method of the request?
  * How did our form do that?  
  * Inspect the form in the browser and see if you can find out how.
1. What is ```patch```?


## DRY (Don't repeat yourself)

### Exercise

What if we want to add a new field when creating/updating a project, we'd have to change it in two places (edit.html.erb and new.html.erb)
Let's clean up our templates.  

* Create a new file ```app/views/projects/_render.html.erb```
* This is known as a partial. (A partial template that is reusable)
* Put the following inside of it:
```erb
  <div class="form-group">
    <%= f.label :name %>
    <%= f.text_field :name, class: 'form-control' %>
  </div>
```
* Change the contents of your edit.html.erb to be:
```erb
  <%= form_for @project do |f| %>
    <%= render "form", f: f %>
    <%= f.submit "Save", class: 'btn btn-success' %>
  <% end %>
```
* Change the contents of your new.html.erb to be:
```erb
  <%= form_for @project do |f| %>
    <%= render "form", f: f %>
    <%= f.submit "Create", class: 'btn btn-success' %>
  <% end %>
```
* In the ```projects_controller.rb``` there are some other areas that can be DRY'd up.
  * Look how ```@project = Project.find params[:id]``` is repeated for **member** actions.
  * Feel free to ask about what I mean when I say **member** actions
* Add the following line to the top of the controller inside the class
```ruby
  before_action :load_project, only: [:show, :edit, :update]
```
* Add the following in the ```protected```
```ruby
  def load_project
    @project = Project.find params[:id]
  end
```
* Remove all other ```@project = Project.find params[:id]``` lines
* Retest all your pages

### Questions

1. What is the second parameter in render do?
  * What if it was written like this? ```<%= render "form", locals: {f: f} %>
1. Why is our form outside of the partial?
1. Why is our submit button outside of the partial?

## Errors

### Exercise

We are going to make sure our Project has a name.  We are going to add validations and display the errors to the user.

* Add the following line to ```app/models/project.rb```
```ruby
  validates :name, presence: true
```
* Create the following view: ```app/views/shared/_errors.html.erb``` (you will have to create the *shared* directory)
```erb
  <% if target.errors.any? %>
    <div class="text-danger">
      <h2><%= pluralize(target.errors.count, "error") %> prohibited this record from being saved:</h2>
      <ul>
        <% target.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
```
* Add the following to the top of ```edit.html.erb``` and ```new.html.erb```
```
  <%= render 'shared/errors', target: @project %>
```
* Try creating and updating a project without a name

### Questions

1. Why did we create a partial in shared, why not in projects?
1. What is target.errors? Where did it come from?
1. Can you reproduce this in the console?  (rails c)
1. Why are there errors?
1. Where are we checking for them?
1. Why does it know what the errors are? How is state preserved after the request? (look at the ```else``` in our ```update``` action)

### ActiveRecord Relationships

Let's add some tasks to these projects!

### Exercise

* Create a new Task model
  * $ rails g model Task name:string description:text project:references
* Look at the migration ```db/migrate/xxxx_create_tasks.rb```
* Look at ```app/models/task.rb```
* Add this line to ```app/models/project.rb``` above our validation:
``` ruby
  has_many :tasks
```
* $ rake db:migrate
* Here's a new one.  Open ```db/schema.rb```
  * That's our database structure! 
*. $ rails c (may need ```bundle exec rails c```)
  * > project = Project.first
  * > project.tasks.create(name: 'Task 1')
  * > task = Task.first
  * > task.project
  * > project.tasks

### Questions

1. What does ```belongs_to``` do again?
1. What does ```has_many``` do?
1. Why are they different and why are they in their respective files?

## Routing Resources

### Exercise

* If we were to have another page for all our tasks (and we will), our routes for tasks would look like this.
```ruby
  get 'tasks', to: 'tasks#index'
  get 'tasks/new', to: 'tasks#new', as: :new_task
  get 'tasks/:id', to: 'tasks#show', as: :task
  post 'tasks', to: 'tasks#create'
  get 'tasks/:id/edit', to: 'tasks#edit', as: :edit_task
  put 'tasks/:id', to: 'tasks#update'
  patch 'tasks/:id', to: 'tasks#update'
```
* This is very repetitive.  There has to be a better way (imagine an infomercial image here)
* There is.  
* These routes are very command for CRUD
* You can replace all your project routes with:
```ruby
  resources :projects
```
* You can replace all your task routes with:
```ruby
  resources :tasks
```
* $ rake routes

I don't like these routes:
* /tasks
* /tasks/:id

I only want to view the tasks inside a project
So I want routes like this:
* /projects/1/tasks
* /projects/1/tasks/1

* All my tasks are children of a project
* So I kinda want my routes nested
* So lets use **nested routes**
* Change the contents of your ```config/routes.rb``` to be
```ruby
  Rails.application.routes.draw do
 
    resources :projects do
      resources :tasks
    end

    root to: 'projects#index'
  end
```
* $ rake routes
* Add the following to ```app/views/projects/show.html.erb```
```erb
  <%= link_to "New Task", [:new, :task], class: 'btn btn-success pull-right' %>

  <table class='table'>
    <thead>
      <tr>
        <th>Name</th>
        <th>Description</th>
        <th></th>
      </tr>
    </thead>
    <tbody>
      <% @project.tasks.each do |task| %>
        <tr>
          <td><%= task.name %></td>
          <td><%= task.description %></td>
          <td><%= link_to "View", task, class: 'btn btn-primary' %></td>
        </tr>
      <% end %>
    </tbody>
  </table>
```
* Click the "New Task" link
  * Oh NO!  

### Questions

1. Why is the second parameter on this ```link_to``` different
1. How come some used _path and some didn't?
1. Can you go back and change them all to not use _path

## STOP HERE

### Advanced topics

* Asset pipeline
* Background jobs
* Interactor
* Services
* Testing

## Next steps (Bonus)

* Create the ```tasks_controller.rb```
* Add the ```new``` action and ```new.html.erb```
  * What does the ```form_for``` look like now that it's nested?
* Add the ```create``` action
  * Use the strong params
* Add the ```update``` and ```create``` actions and the ```edit.html.erb``` template
* Don't forget to make a ```_form.html.erb```
* We haven't done any deleting.  Can you figure out how to delete?
* Create the ```Developer``` model
* Associate a ```Developer``` to a task in ActiveRecord
* Add a ```collection_select``` to your ```app/views/tasks/_form.html.erb```

## Next next steps (2x Bonus)

* Add a keyword filter to you ```projects#index``` aka ```app/views/projects/index.html.erb```
  * This is a form that submits a GET to the index action
  * Queries projects for ```projects.name LIKE ?, '%#{keyword}#'```
* Add search filters to your task list (filter by developer)

## Next next next steps (3x Bonus)

* Add the gem ```aasm``` (Acts as state machine)
* Add a state to ```task.rb```
* Allow a task to have the states: [:new, :in_progress, :completed]
* Add search filters to your task list (filter by keyword, state)





















