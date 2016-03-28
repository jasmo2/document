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
Comparing **Ruby**'s flexibility and OOP structure shines over to with strong type languages like **Java** or **C\+\+**. In **Ruby** everything is an object, even numbers. Making an easy way to interact between objects through messages, because they share a common interface.

It is not a secret that **Ruby** is an awesome language that makes developers "happy code". Moreover, its' web-framework Ruby on Rails (*RoR*) makes code, even faster. But, not everything is perfect, its major weakness come from its major enhance, the soft type and lack strict interfaces, giving the perfect recipe  to spaghetti code. But,  modern web-frameworks' principle is [convention over configuration](http://en.wikipedia.org/wiki/Convention_over_configuration) that step over a developers preferences. Additionally, the magic that happened behind the scenes given by different **Ruby**  frameworks lead to misuse them.

Listed below the are most common mistakes on **Ruby** and its most popular web-framework **RoR**.


## Using Logic Inside The Views
Nowadays for web development the most common pattern is the MVC  ( [Model View Controller](http://www.tutorialspoint.com/design_pattern/mvc_pattern.htm) ). Besides,  beginners tend to code business logic in the View layer because  helper methods and template engines allow to query directly the database.

On the example below the template engine *ERB* (Embedded RuBy) has been used to demonstrate our point.

> app/views/courses/show.erb.html

```ruby
...
<% if @course.present? %>
	<% @top_student = @course.student.order('rank DESC').first %>
	<%= link_to "Top student", @top_student %>
<% end %>
...
```

An appropriate way to write this chunk of code is move this logic down the controller, where the top student can be query, and presented on a cleaner way to the view layer.

> app/controllers/courses_controllers.rb

```ruby
def show
	@course = Course.find(params[:id])
	@top_user = @course.student.order('rank DESC').first
end
```

Now the view will be cleaner and easier to read:

> app/views/courses/show.erb.html

```ruby
<% if @top_student.present? %>
	<%= link_to "Top student", @top_student %>
<% end %>
```

Even though,  query on the view gives the sense that something could still refactor, and modules come to the rescue by encapsulating *necessary*  logic presented on view layers.

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
These are the default methods given by the **RoR**'s scaffold . However, it does not mean these are the only ones could be written on the controller. A typical example could be `users_reset_password method` inside the users_controller, which helps recover the user secret password if they lost it.

For instance, a bunch of methods inside the controller means that it should be refactor into smaller controllers :

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

Assigning complex logic inside the controller layer usually breaks the DRY principle by repeating a concern across multiple controllers. Instead, these kind of logic should be down into the model layer helping reusing the method on multiple controllers and making it more easy to maintain.

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

On the code above, calling an object several times on the same controller gives us the sense of *code smell* meaning it could be refactor; next is presented an alternative:


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
Is a common practice to use conditionals on an application, they are use to choose how a certain actions should behave, if one way or another. Although, sometimes there are to many of them in one method; giving an alert sign of *smelly code* ([How do I smell Ruby code?](http://rubylearning.com/blog/2011/03/01/how-do-i-smell-ruby-code/)) asking to refactor right away.

For instance, lets take the next piece of code as an example:


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
These previous code shows a news paper application.

The user's controller has several roles and each one redirects to an specific *path* after go into the *users_controller#index* .
The problem here is the way is written, because,  is to messy to read, misuse the *Ruby* power, but overall is hard to maintain.

> After refactor

> app/controllers/users_controller.rb


```ruby

redirection_path = Hash.new{|hash,key| hash[key] = user_path}
redirection_path[:admin] =  admin_path
redirection_path[:publisher] =  publisher_path
redirection_path[:subscriber] =  subscriber_path

@document.save ? redirect_to redirection_path[current_user.role.to_sym]  :  (render :new)
```

On the previous refactor the Logic Hash helps to redirect the user with a clean syntax helping the maintenance of the application .

## Not all models should be *ActiveModels*
**RoR** give us the sensation  the only types of models, are the one provided by ActiveRecord, but is is a Lie.
Usually in projects, are heavy logic models that break the *single responsibility principle* ([*SOLID*](https://vimeo.com/12350535)) giving tasks that do not belongs to the model itself.


On the example below, will refer to a blog application:



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

> After refactor

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

On the controller, the code will look clean and the variation will take place by calling different objects and not just one:


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



## ORM memory overloading queries
**RoR** is provided by default by with many tools like **ActiveRecord**, but the "magic" some times have performance issues if they are not used well.
On large applications memory consumption and response times could be an issue ([How to optimize Active Record Queries](http://codebeerstartups.com/2013/04/how-to-optimize-active-record-queries/)).


For instance, there are many tips that will help you to make database queries as smooth as possible.

+ **Sumatories**: better is to past the symbol and NOT a reference variable.
	* Transaction.sum(&:user_id)
	* Transaction.sum(:user_id)

	By passing a reference variable **Ruby** will create an instance for each selected row; on the other hand, the second option leave the heavy to the database.

+ **Uniq**: many times a query only want to ask for the distinct values on a table.
	*Transaction.pluck(:user_id).uniq
	*Transaction.uniq.pluck(:user_id)

	The methods above are the same and create the same result but the difference resides on how the query is done. The first one the column values, after that it filter. But the second one executes the *DISTINCT* over the database, which means better performance.

+ **Plunk vs Map**: these methods help to get specific columns from the records on the *DB*
	*Transaction.all.map(&:user_id)
	*Transaction.pluck(:user_id)

	With the *Map* helper **Ruby** will instantiate a model for each row the query retrieve, which translates on consuming time. Contrasting with *Pluck* it only generates an array with the results of the column is require.

+ **find_each vs. each**: Time to time when a query is done a post process need to be perform. In case the retrieve quantity is small is ok to use **each**; once the result become massive, is better to use **find_each**, because, it set the result on batches of 1000, making a lower load to the memory. In case you need a whole update on a table also is recommended to use **update_all**, this method will improve upto 100x the performance in contrast to doing it one by one.



## How Background task could save you application from blocking.
Even though, an application can accomplish high performance queries to the database, there are tasks that consume time. An example is an application that need to process images.
Here a client will upload an image and the app will convert the format to png and resize to AxB regardless the input format and/or image-size.

Below is presented the blocking app. In this case the application waits until the process is finish to continue the application flow.

![Blocking APP][blocking_app]

To avoid the problem is good idea to use background queue jobs. These are the ones in charge to execute the task separate to the main application. Fortunately, ruby has some gems, which makes background jobs extremely easy, for example:

* [Sidekiq](http://sidekiq.org/)
* [Shoryuken](https://github.com/phstc/shoryuken)
* [Delay Job](https://github.com/collectiveidea/delayed_job)

In this case the application will have a new structure that look like the image below.

![non blocking APP][non_blocking_app]


Using background jobs could help you to avoid headaches trying to improve times on CPU consuming processes.

## Gems everywhere
The main idea of the **rubygems**  ([rubygems.org](https://rubygems.org/)) is to do not re-invent de wheel, in many cases you could use a specific library that add functionality to the project need. On the other hand, the dark side by depending too much ok 3º libraries is that programmers do not take time to learn how the implementation actually works this could cause that modify the functionality could be an difficult task or even impossible.

### Code sizing
With each third party **gem** used on an application the overhead will increase, the response time will grow and project''s size will get bigger in unexpected dimensions, because each **gem** comes with extra files.


### Bundle update, incompatibilities
On a **Ruby** project updating the dependencies is a common task that every project should do, to keep the security and performance at the best quality. But, sometimes when you have a bunch of dependencies a simple update could become a nightmare.

Imagine the case where your project has two **gems** but at the same time they relies on third **gem**  but with different versions. On this case, monkey patch one of the gems could be a solution, but the best way to sort the problem is to code your own implementation to solve the problem. It could take around 20 to 30 minutes to understand how a gems works, before beginning the implementation.

### Remember
Remember before add a new **gem** to an application, ans the following questions:
+ Does it provide the functionality I need?
+ Can I implement the functionality without the gem and is simple enough for me?
+ Does the overhead provided by **gem** worth functionality?
+ Have an active maintainer?


### Useful *gems*
#### Below we provide a list of useful gems and their functionality
***

+ [Devise](https://github.com/plataformatec/devise); *Devise is a flexible authentication solution for Rails based on Warden*
+ [Omniauth](https://github.com/intridea/omniauth); *OmniAuth: Standardized Multi-Provider Authentication*
+ [CarrierWave](https://github.com/carrierwaveuploader/carrierwave); *This gem provides a simple and extremely flexible way to upload files from Ruby applications. It works well with Rack based web applications, such as Ruby on Rails.*
+ [Kamari](https://github.com/amatsuda/kaminari); *A Scope & Engine based, clean, powerful, customizable and sophisticated paginator for modern web app frameworks and ORMs*
+ [Puma](https://github.com/puma/puma); *Puma is a simple, fast, threaded, and highly concurrent HTTP 1.1 server for Ruby/Rack applications.*
+ [Think Sphinx](https://github.com/pat/thinking-sphinx); *Thinking Sphinx is a library for connecting ActiveRecord to the Sphinx full-text search tool*
+ [MongoId](https://github.com/mongodb/mongoid); *Mongoid is an ODM (Object-Document-Mapper) framework for MongoDB in Ruby.*
+ [Rspec](https://github.com/rspec/rspec); *Behaviour Driven Development for Ruby*
+ [Capybara](https://github.com/jnicklas/capybara); *Capybara helps you test web applications by simulating how a real user would interact with your app.*
+ [Factory Girl](https://github.com/thoughtbot/factory_girl); *factory_girl is a fixtures replacement with a straightforward definition syntax, support for multiple build strategies ...*
+ [Capistrano](https://github.com/capistrano/capistrano/); *Capistrano is framework for building automated deployment scripts*
+ [Sidekiq](https://github.com/mperham/sidekiq); *Simple, efficient background processing for Ruby.*
+ [Bundler](https://github.com/carlhuda/bundler); *Bundler makes sure Ruby applications run the same code on every machine.*
+ [Ruby-PG](https://github.com/ged/ruby-pg); *Pg is the Ruby interface to the PostgreSQL RDBMS.*
+ [Active Merchant](https://github.com/activemerchant/active_merchant); * Shopify's requirements for a simple and unified API to access dozens of different payment gateways with very different internal APIs was the chief principle in designing the library.*
+ [RMagick](https://github.com/rmagick/rmagick); *RMagick is an interface between the Ruby programming language and the
ImageMagick image processing library.*
+ [Rails](https://github.com/rails/rails); *Rails is a web-application framework that includes everything needed to create database-backed web applications according to the Model-View-Controller (MVC) pattern.*
+ [Syro](https://github.com/soveran/syro); *Simple router for web applications.*
+ [CanCanCan](https://github.com/CanCanCommunity/cancancan); (the original gem could be find on *[CanCan](https://github.com/ryanb/cancan)*) *CanCan is an authorization library for Ruby on Rails which restricts what resources a given user is allowed to access.*
+ [Haml](https://github.com/haml/haml); *It's designed to make it both easier and more pleasant to write HTML documents, ...*



## Sensitive data into repositories
##### [Sensitive data in git repositories](http://blog.arvidandersson.se/2013/06/10/credentials-in-git-repos).
The previous article gives us an idea of how the important data like cloud credentials and/or databases credentials should be keep out of git scope by adding them to *__.__gitignore* file. Although, **RoR** provides *environment variables* to help protect sensitive data, people get used to rails "magic", which makes them forget to ignore sensitive data files and leave to security flaw open in your repository. Keep in mind code leave on public repositories, open the gates to the attacker to get all kind of information that will compromise your application.

The usual files that need to be ignore on an **RoR** project are:
+ config/initializers/secret_token.rb
+ config/secrets.yml
+ config/database.yml



## Testing, Whats that for?
The ruby community is well known for write automated testing. But many rookies does not use them; it does not matter if is TDD or BDD, or even testing after the code has been written. **Test musT be written**.[red/green/refactor](https://vimeo.com/43734265)

Continuing, with the test modality the next example will show a simple way how an application could be written:
In our application is a slugifier for the *Spanish* language that will replace any special character to the equivalent in english.

The next are the assumptions on the example:
+ Ruby >= 2.2.x
+ RSpec gem to test.
+ Test are going to be written first
+ Then code.

> test.rb

```ruby
require './slugifier'


RSpec.describe "slugifier function"  do
	it 'should return lowercase always' do
	    big_letters = "CapitalLettersTest"
	    result = slugifier(big_letters)
        expect(result).to eq("capitalletterstest")
	end
	it 'should return no accents' do
	    acents = "áéíóúñaeiou"
	    result = slugifier(acents)
        expect(result).to eq("aeiounaeiou")
	end
	it 'should return - instead of "white-spaces"' do
	    acents = "spaces test"
	    result = slugifier(acents)
        expect(result).to eq("spaces-test")
	end
	it 'should return vocals as well as consonants' do
	    strung = "a e i o u x z y"
	    result = slugifier(strung)
        expect(result).to eq("a-e-i-o-u-x-z-y")
	end
	it 'should strip all non-alphanumeric' do
	    non_alphanumeric = "@lpha #merics 1 test"
        expect(slugifier(non_alphanumeric)).to eq("lpha-merics-1-test")
	end
end
```
The test below is going to to check. functionalities require to correct run slugifier application.
First is going to check if the program could convert capcase to lowercase. Then if it special character on the result have disappear, after that check the spaces on the input string will be replace for *-*. Finally, is going to strip the non alphanumeric characters on the string

At first the result will throw the following

```log
5 examples, 5 failures

Failed examples:

rspec ./slugifier_spec.rb:4 # slugifier function should return lowercase always
rspec ./slugifier_spec.rb:9 # slugifier function should return no accents
rspec ./slugifier_spec.rb:14 # slugifier function should return - instead of "white-spaces"
rspec ./slugifier_spec.rb:19 # slugifier function should return vocals as well as consonants
rspec ./slugifier_spec.rb:24 # slugifier function should strip all non-alphanumeric
```
To get green the application should ensure the five requirements, the following code is an proximation to solve the problem.

```ruby
def slugifier(strings)
	slug = strings.downcase.gsub(/á+/,'a')
		.gsub(/é+/,'e')
		.gsub(/í+/,'i')
		.gsub(/ó+/,'o')
		.gsub(/ú+/,'u')
		.gsub(/ñ+/,'n')
		.gsub(/[^\w\s-]/,'')
		.gsub(/\s+/,'-')
end
```

After the implementation everything is back green and the application is fully functional now. On more complex scenarios where not just application''s functionalities are going to be test, but also user interaction the previous **gem RSpec and Capybara** is great choice, because it allow to click and wait for visual answer if there is one. Finally, remember to write test on your applications to give continuity and help add new features easily.



## Realize that Ruby is actually a programming language
Most developers that start with **Ruby** is because **RoR** makes their life too easy (Like my case *XD* ), and most of them believe that the framework is actually the programming language. Normally, people realize that rails is just a way to do web applications, but nowadays with microservices boom is common to find minimalistic web-frameworks like:
+ [satz](https://github.com/syro/satz), also refer to  *[syro](http://soveran.github.io/syro/)*
+ [synatra](http://www.sinatrarb.com/)
+ [nancy](http://guilleiguaran.github.io/nancy/)
+ [grape](http://www.ruby-grape.org/)

The odd part of working with minimalistic frameworks is that most common rails app defaults are not added. Moreover, the developer should be more experience to code on these types of frameworks vs **RoR** , once code on a minimal framework most developers stay with them, because they are much faster than **RoR** ((ruby frameworks benchmark)[http://www.sitepoint.com/ruby-microframeworks-round/]), also learn how to integrate all the gem ecosystem to its new workshop.



## Final thoughts
**Ruby** is a great language that is fully **OOP**;  however, most of the people get into it through **RoR**, which is a great start. Is important to keep in mind the good practices and the common mistakes done by rookie developers.

Finally, remind that **Ruby** is a complete programming language that allow the user to create big extensible applications, that not necessary need to be part of an all-in-one framework, but also could code as small distributed microservices.

[blocking_app]: https://raw.githubusercontent.com/jasmo2/document/master/images/blocking_app.png "title"
[non_blocking_app]: https://raw.githubusercontent.com/jasmo2/document/master/images/non_blocking_app.png "title"
