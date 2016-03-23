<!-- #Questions
Ruby 10_Q

NodeJS 10_Q


2easy; 2medium; 6-exp

top 10 ruby most common mistakes
out line -> Title paragraph
	why is popular
	challenges
	10 subtitltles
1500 words -->

# Top 10 Most Common Ruby Mistakes
Comparing **Ruby**'s flexibility and OOP structure shines over to with strong type languages like **Java** or **C\+\+**. Due to, everything is an object, even numbers, creates a common interface which is the way they interact between each other through messages.

It is not a secret that **Ruby** is an awesome language that makes developers "happy code". Moreover, its' web-framework Ruby on Rails (*RoR*) makes code, even faster. But, not everything is perfect, its major weakness come from its major enhance, the soft type and lack strict interfaces, giving the perfect recipe  to spaghetti code. But,  modern web-frameworks' principle is [convention over configuration](http://en.wikipedia.org/wiki/Convention_over_configuration) that step over a developers preferences. Additionally, the magic that happened behind the scenes given by different **Ruby** frameworks lead to misuse them.

Then next items are the common mistakes on **Ruby** and its most popular web-framework **RoR**.


## Using Logic Inside The Views
The most common patterns now a days for web development is the MVC  ( [Model View Controller](http://www.tutorialspoint.com/design_pattern/mvc_pattern.htm) ) pattern. But as beginners people tend to code business logic in the View layer. Normally, occurs because some helper methods and template engines allow to query directly the database.

> app/views/courses/show.erb.html

```ruby
<% if @course.present? %>
	<% @top_student = @course.student.order('rank DESC').first %>
	<%= link_to "Top student", @top_student %>
<% end %>
```

A appropriate way to write these chunk of code is move this logic down the controller, where the top student can be query, and presented cleaner on the view layer.

> app/controllers/courses_controllers.rb

```ruby
def show
	@course = Course.find(params[:id])
	@top_user = @course.student.order('rank DESC').first
end
```

Now the view will look better:

> app/views/courses/show.erb.html


```ruby
<% if @top_student.present? %>
	<%= link_to "Top student", @top_student %>
<% end %>
```

Even though,  query on the view gives the sense that something could still refactor, and modules come to the rescue by encapsulating *necessary* view logic.

> app/helpers/course_helper.rb

```ruby
module CoursesHelper
    def top_student(student)
			if student.present?
        	link_to "Top student", student
    	end
    end
end

```
Finally, the view will be as clean as possible and just responsible to to render html.


> app/views/courses/show.erb.html


```ruby
<%= top_student(@top_student) %>
```

## Fat Controllers
The problems in the controller are divide in two big groups *many methods *inside one controller and/or *too much logic* inside the view layer.

### Too much methods
In the MVC pattern the *C* references the controller, it is the one in charge to communicate the business logic with the layout, and it should have around **7 methods**:

> app/controllers/users_controller.rb

```ruby
def index
end

def show
end

def new
end

def create
end

def update
end

def edit
end

def delete
end
```
These are the default methods given by the **RoR**''s scaffold .But, it does not mean these are the only ones you could write on the controller. A typical example could be `users_reset_password method` inside the users_controller which helps recover the user secret password if they lost it.

In contrast, a bunch of methods inside the controller means that it should be refactor into smaller controllers, for instance:

> Before refactor


> app/controllers/users_controller.rb


```ruby
## users scaffold
def user_password_get
end
def user_password_reset
end
def user_password_generate
end
def user_password_remove
end
```
> After refactor

users_controller will be on its own file with the CRUD methods. And `passwords_controllers.rb` will contain only the password specific methods.

> app/controllers/passwords_controller.rb

```ruby
def new
end
def create
end
def update
end
def edit
end
def destroy
end
```

Now the controller is concise enough to handle a process in the application.


### Controllers essential logic.

Assigning complex logic inside the controller layer usually breaks the DRY principle by repeating a concern across multiple controllers. Instead, these kind of logic should be down into the model layer.

> Before refactor

> app/controllers/items_controller.rb

```ruby
def publish
 if @item.is_approved?
  	@item.published_on = Time.now
  		if @item.save	&& @item.user != 'master user'
        flash[:notice] = "Your item published!"
      else
        flash[:notice] = "There was an error."
      end
    	else
      	flash[:notice] = "There was an error."
			end
    redirect_to @item
	end
end
```

For instance, calling an  object several times on a single controller gives us the sense on code smell meaning it could be refactor; next is presented al alternative:


> After refactor

> app/controllers/items_controller.rb

```ruby
def publish
  if @item.publish
    flash[:notice] = "Your item published!"
  else
    flash[:notice] = "There was an error."
  end
  redirect_to @item
end

```

> app/models/items.rb
```ruby
def publish
  if !is_approved? || user == 'master user'
    return false
  end
  self.published_on = Time.now
  self.save
end
```

After refactoring, the publish method is push down into the model layer, as is shown now it could be access in the any controller with out repeating the code in multiple parts of the project.

## To much conditionals.
Normally, developers using conditionals filter the application to go one way or another.But, sometimes to many of them in one method is a sign of **smelly code** ([How do I smell Ruby code?](http://rubylearning.com/blog/2011/03/01/how-do-i-smell-ruby-code/)), and they could be refactor.

For instance, let''s take the next piece of code as an example:


> app/controllers/users_controller.rb

```ruby
  def index
	 if @document.save
		if current_user.role == "admin"
			redirect_to admin_path
		elsif current_user.role == "publisher"
		 redirect_to publisher_path
		elsif current_user.role == "subscriber"
		 redirect_to subscriber_path
		else
		 redirect_to guest_path
		end
	 else
	   render :new
	 end
 end
```
These previous code shows a news paper application. On the user model has several roles and each one redirects to an specific method after go into the *index* .
The problem here is the way is written, because,  is to messy to read and misuse the *Ruby* power, but overall is hard to maintain.

> After refactor

> app/controllers/users_controller.rb


```ruby

redirection_path = Hash.new{|hash,key| hash[key] = user_path}
redirection_path[:admin] =  admin_path
redirection_path[:publisher] =  publisher_path
redirection_path[:subscriber] =  subscriber_path

@document.save ? redirect_to redirection_path[current_user.role.to_sym]  :  (render :new)
```

On the previous refactor the Logic Hash helps to redirect the user with a clean maintenance syntax.

## Not all models should be *ActiveModels*
**RoR** give us the sensation  the only types of models, are the one provided by ActiveRecord, but is is a Lie.
Usually in projects are heavy logic models that break the *single responsibility principle* ([*SOLID*](https://vimeo.com/12350535)). Moreover, gives responsibility to which do not belongs to the model itself.


On the example below, a the user_model will be from a blog application:



> app/models/user.rb

```ruby
class User < ActiveRecord::Base
  has_many :posts
  has_many :reviews

  def cancel!
	    self.class.transaction do
			transactions ensure all enclosing database operations are atomic
			      self.update!(is_approved: false)
			      self.posts.each { |post| post.update!(is_approved: false) }
			      self.reviews.each { |review| review.update!(is_approved: false) }
			end
		end
end
```

On the model above we could see that the user model has not just the responsibility to update itself, but also it has to do transaction to the posts and reviews models. Now our model is an anti-pattern known as **God Object**. A much better way to a handle these other responsibilities.

> Refactor
> app/models/user_cancelation.rb

```ruby
class UserCancelation
	def initialize(user)
		@user = user
	end

	def create
		@user.class.transaction do
			cancel_user!
			cancel_items!
			cancel_reviews!
		end
	end

	private
		def cancel_user!
			self.update!(is_approved: false)
		end
		def cancel_posts!
			self.posts.each { |post| post.update!(is_approved: false) }
		end
		def cancel_reviews!
			self.reviews.each { |review| review.update!(is_approved: false) }
		end
end
```

As is shown above, the *UserCancelation* model is in charge to denied the access to an specific user as well as un-approve the posts and reviews. Now, the concerns are separate on different objects, making easier to change on the future. Is implemented as a **PORO** (Plain Old Ruby Object).

On the controller, the code will look clean and the variation will take place on calling different objects and not just one:


> app/controller/users_controller.rb

```ruby
class UsersController < ApplicationController
  def cancel_subscription
    @user = User.find(params[:id])
    cancelation = UserCancelation.new(@user)
    cancelation.create!
    redirect_to @user, notice: 'Successfully suspended.'
	end
end
```

As shown on the refactor is better to separate the concerns on different models. It could be an excellent approach when the load on a single method want to be distributed into smaller tasks.



~~Ruby is originally concede to **OOP** but **RoR** is a MVC-pattern oriented Ruby programming.~~


## ORM memory overloading queries
**RoR** is provided by default by with many tools like ActiveRecord, but the "magic" some times have performance issues if they are not used well.
On large applications memory consumption and response times could be an issue ([How to optimize Active Record Queries](http://codebeerstartups.com/2013/04/how-to-optimize-active-record-queries/)).

## How Background task could save you application from blocking.
Even though the application accomplish high performance queries to the database, there are tasks that consume time. An example is an application that need to process images.
Here a client will upload an image and the app will convert the format to png and resize to 80x60 regardless the input format and/or image-size.

Below is presented the blocking app. In this case the application waits until the process is finish to continue the application flow.

![Blocking APP][blocking_app]

To avoid the problem is good idea to use background queue jobs. These are the ones in charge to execute the task separate to the main application. Fortunately, ruby has some gems, which makes background jobs extremely easy, for example:

* [Sidekiq](http://sidekiq.org/)
* [Shoryuken](https://github.com/phstc/shoryuken)
* [Delay Job](https://github.com/collectiveidea/delayed_job)

In this case the application will have a new structure that look like the image below.

![non blocking APP][non_blocking_app]



## Gems everywhere
The main idea of the **rubygems**  ([rubygems.org](https://rubygems.org/)) is to do not re-invent de wheel, in many cases you could use a specific library that add functionality to the project need. On the other hand, the dark side by depending too much ok 3º libraries is that programmers do not take time to learn how the implementation actually works. Additionally, ...




## Sensitive data into repositories
##### [Sensitive data in git repositories](http://blog.arvidandersson.se/2013/06/10/credentials-in-git-repos).
The previous article gives us an idea of how the important data like cloud credentials and/or databases credentials should be keep out of git scope by adding them to *__.__gitignore* file. Although, **RoR** provides *environment variables* to help protect sensitive data, people get used to rails "magic", which makes them forget to ignore sensitive data files and leave to security flaw open in your repository.


## Testing, Whats that for?
The ruby community is well known for write automated testing. But many rookies does not use them; it does not matter if is TDD or BDD, or even testing after the code has been written. **Test musT be written!**

## Realize that Ruby is actually a programming language
Most developers that start with **Ruby** is because **RoR** makes their life too easy (Like my case *XD* ), and most of them believe that the framework is actually the programming language. Normally, people realize that rails is just a way to do web applications, but now a days with microservices boom is common to find minimalistic web-frameworks like:
+ [satz](https://github.com/syro/satz), also refer to  *[syro](http://soveran.github.io/syro/)*
+ [synatra](http://www.sinatrarb.com/)
+ [nancy](http://guilleiguaran.github.io/nancy/)
+ [grape](http://www.ruby-grape.org/)

The odd part of working with minimalistic frameworks is that most common rails app defaults are not added. Moreover, the developer should be more experience to code on these types of frameworks vs **RoR** .


## Final thoughts
**Ruby** is a great language that is fully **OOP**.However, most of the people get into it through **RoR**, which is a great start. Is important to keep in mind the good practices and the common mistakes done by rookie developers.

Finally, remind that **Ruby** is a complete programming language that allow the user to create big extensible applications, that not necessary need to be part of an all-in-one framework, but also could code as small distributed microservices.

[blocking_app]: https://raw.githubusercontent.com/jasmo2/document/master/blocking_app.png "title"
[non_blocking_app]: https://raw.githubusercontent.com/jasmo2/document/master/non_blocking_app.png "title"
