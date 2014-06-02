# Rotten Mangoes

Let's get going on our second Rails app!

We're going to build a Rotten Tomatoes clone called Rotten Mangoes. This app will be our first dive into several crucial Rails concepts that will serve you well into the future:
* associations
* validations/errors
* “business logic” (it's [business time](http://www.youtube.com/watch?v=WGOohBytKTU))
* partials
* helpers
* authentication from scratch
* authorization
* flash alerts and notices
* helper methods
* nested resources
* filters
* and more! (jk that's probably all of them)

Again, this tutorial is divided into handy [commits](https://github.com/lighthouse-labs/rotten_mangoes/commits/master). Each section correlates with a commit, and you should commit at the completion of each section.

## 1. rails new [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/70b16a6829239ea216c50bc987d7f3f72562cb53)

Ah, new beginnings. A fresh start. A clean slate. It's a nice feeling, isn't it?

## 2. generating movie model [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/cc0763afca78ab801acffb97cacac685f08b65e0)

Let's start by generating the model representing the focus of our app: movies! From the console:

    rails g model movie title director runtime_in_minutes:integer description:text poster_image_url release_date:datetime

Notice some of our columns have no data type specified. The default type is string, so if omitted, Rails will automatically assign string to the appropriate column.

`rake db:migrate` this thing and let's keep on chugging!

## 3. generating movies controller and views [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/e86479fa4bb05165623e69d26b137dcc15693521)

I'm not gonna lie: this is a big commit. But I trust you'll zoom through it. We're simply setting up some standard CRUD, Rails-style. Use Lighthouse Forum as a reference, and let's try to set up the following before continuing the tutorial. I think you got this!
* index should show all movies
* show should show one movie
* new/create should work together to let you add a movie
* edit/update should work together to let you edit a movie
* destroy should, well, destroy the movie

Okay, how'd it go? Let's compare what you've done with how I've worked through this.

First, I ran this from the console:

    rails g controller movies index show new edit

This generates routes, controller actions, and views for me, but I want to refactor the routes right away. In `routes.rb`:

    RottenMangoes::Application.routes.draw do
      resources :movies
    end

Your `MoviesController` should look very familiar. No surprises here:

    class MoviesController < ApplicationController

      def index
        @movies = Movie.all
      end

      def show
        @movie = Movie.find(params[:id])
      end

      def new
        @movie = Movie.new
      end

      def edit
        @movie = Movie.find(params[:id])
      end

      def create
        @movie = Movie.new(movie_params)

        if @movie.save
          redirect_to movies_path
        else
          render :new
        end
      end

      def update
        @movie = Movie.find(params[:id])

        if @movie.update_attributes(movie_params)
          redirect_to movie_path(@movie)
        else
          render :edit
        end
      end

      def destroy
        @movie = Movie.find(params[:id])
        @movie.destroy
        redirect_to movies_path
      end

      protected

      def movie_params
        params.require(:movie).permit(
          :title, :release_date, :director, :runtime_in_minutes, :poster_image_url, :description
        )
      end

    end

Let's take a look at the views. `movies/index.html.erb` first:

    <h1>Rotten Mangoes</h1>
    <%= link_to "Submit a movie!", new_movie_path %>
    <hr>
    <% @movies.each do |movie| %>
      <%= link_to image_tag(movie.poster_image_url), movie_path(movie) %>
      <h2><%= link_to movie.title, movie_path(movie) %></h2>
      <h3><%= movie.release_date %></h3>
      <h4>Dir. <%= movie.director %> | <%= movie.runtime_in_minutes %> minutes</h4>
      <p><%= movie.description %></p>
      <hr>
    <% end %>

Nothing too different from our forum app so far. `movies/show.html.erb` too:

    <%= link_to "Back to all movies", movies_path %><br/>

    <%= link_to image_tag(@movie.poster_image_url), movie_path(@movie) %>
    <h2><%= @movie.title %> (<%= link_to "edit", edit_movie_path(@movie) %>, <%= link_to "delete", movie_path(@movie), method: :delete, confirm: "You sure?" %>)</h2>
    <h3><%= @movie.release_date %></h3>
    <h4>Dir. <%= @movie.director %> | <%= @movie.runtime_in_minutes %> minutes</h4>
    <p><%= @movie.description %></p>

Our `movies/new.html.erb` is pretty straightforward as well:

    <%= link_to "Back to all movies", movies_path %>

    <h2>Submit a movie!</h2>
    <%= form_for @movie do |f| %>
      <div>
        <%= f.label "title" %><br>
        <%= f.text_field "title" %>
      </div>
      <div>
        <%= f.label "release_date" %><br>
        <%= f.date_field "release_date" %>
      </div>
      <div>
        <%= f.label "director" %><br>
        <%= f.text_field "director" %>
      </div>
      <div>
        <%= f.label "runtime_in_minutes" %><br>
        <%= f.number_field "runtime_in_minutes" %>
      </div>
      <div>
        <%= f.label "poster_image_url" %><br>
        <%= f.text_field "poster_image_url" %>
      </div>
      <div>
        <%= f.label "description" %><br>
        <%= f.text_area "description" %>
      </div>
      <div>
        <%= f.submit "Submit" %> 
      </div>
    <% end %>

Notice we have a new [form helper](http://guides.rubyonrails.org/form_helpers.html) method (in fact, it's new to Rails 4!) called date_field.

Our `movies/edit.html.erb` is almost identical:

    <%= link_to "Back to this movie", movie_path(@movie) %>

    <h2>Edit this movie.</h2>
    <%= form_for @movie do |f| %>
      <div>
        <%= f.label "title" %><br>
        <%= f.text_field "title" %>
      </div>
      <div>
        <%= f.label "release_date" %><br>
        <%= f.date_field "release_date" %>
      </div>
      <div>
        <%= f.label "director" %><br>
        <%= f.text_field "director" %>
      </div>
      <div>
        <%= f.label "runtime_in_minutes" %><br>
        <%= f.number_field "runtime_in_minutes" %>
      </div>
      <div>
        <%= f.label "poster_image_url" %><br>
        <%= f.text_field "poster_image_url" %>
      </div>
      <div>
        <%= f.label "description" %><br>
        <%= f.text_area "description" %>
      </div>
      <div>
        <%= f.submit "Submit" %> 
      </div>
    <% end %>

Phew! How'd you do? Most of your apps will involve at least some of this, so getting quick at basic Rails CRUD is an important skill for the toolbelt.

## 4. refactoring movies form into partial [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/de5adfb7e56c5b350821e80343ad8318c671b4e8)

In our last commit, I hope that DRY-dee sense tingled while cutting and pasting that `form_for`. Good! Rails' solution for this is [partials](http://guides.rubyonrails.org/layouts_and_rendering.html#using-partials).

So let's `touch app/views/movies/_form.html.erb` and extract our form from `edit.html.erb` and `new.html.erb` right in there:

    <%= form_for @movie do |f| %>
      <div>
        <%= f.label "title" %><br>
        <%= f.text_field "title" %>
      </div>
      <div>
        <%= f.label "release_date" %><br>
        <%= f.date_field "release_date" %>
      </div>
      <div>
        <%= f.label "director" %><br>
        <%= f.text_field "director" %>
      </div>
      <div>
        <%= f.label "runtime_in_minutes" %><br>
        <%= f.number_field "runtime_in_minutes" %>
      </div>
      <div>
        <%= f.label "poster_image_url" %><br>
        <%= f.text_field "poster_image_url" %>
      </div>
      <div>
        <%= f.label "description" %><br>
        <%= f.text_area "description" %>
      </div>
      <div>
        <%= f.submit "Submit" %> 
      </div>
    <% end %>

Nice! Now all we have to do is delete that form from both `new.html.erb` and `edit.html.erb`, and replace it with this:

    <%= render 'form' %>

I hope you're paying attention to all the naming conventions and quirks. Partial _files_ begin with an underscore, but when you're rendering them, you can omit it.

## 5. adding movie validations [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/a3148a3af8e0c0a97b05691186710f9581548d4b)

To make sure our data stays clean despite users using our forms, we should write some [validations](http://edgeguides.rubyonrails.org/active_record_validations.html) in `movie.rb`:

    class Movie < ActiveRecord::Base

      validates :title,
        presence: true
     
      validates :director,
        presence: true
     
      validates :runtime_in_minutes,
        numericality: { only_integer: true }
     
      validates :description,
        presence: true

      validates :poster_image_url,
        presence: true

      validates :release_date,
        presence: true

      validate :release_date_is_in_the_future

      protected
     
      def release_date_is_in_the_future
        if release_date.present?
          errors.add(:release_date, "should probably be in the future") if release_date < Date.today
        end
      end

    end

Notice how the last validation uses `validate` instead of `validates` and specifies a method name to use for performing a custom validation. This validation checks to make sure the movie's release date is in the future. It only performs the check if the release_date attribute is `present?` (opposite of `blank?`) since there is already a presence validation on this field.

## 6. displaying validations errors in movies form [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/e7af5af5efb568b89f6b5f9525b190bf85add326)

If you try to save an invalid movie, it doesn't let you. But it doesn't list all the errors out to you. That's lame. Let's fix that!

Since we want the errors to be shown when someone is updating or creating a movie, and since the _form partial is being used in both cases, it's a good logical place to put this. 

Remember our `.errors.full_messages` from using ActiveRecord before? We'll just go through each error message and display it as part of a bulleted list:

    <%= form_for @movie do |f| %>
      <% if @movie.errors.any? %>
        <div>
          <%= pluralize(@movie.errors.count, "error") %> prevented this movie from being submitted:
          <ul>
            <% @movie.errors.full_messages.each do |msg| %>
              <li><%= msg %></li>
            <% end %>
          </ul>
        </div>
      <% end %>
      [...]
    <% end %>

## 7. generating user model (has_secure_password) [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/fdba22ed8d34213598ca89d9b5a6964326c00b9c)

Next we can support for users to the app. Users will be able to register and login to the site. In Rails, there are multiple options and strategies to implementing authentication. We'll be exploring a simple yet popular approach: `has_secure_password`. See Rails docs for has_secure_password [here](http://api.rubyonrails.org/classes/ActiveModel/SecurePassword/ClassMethods.html#method-i-has_secure_password).

[This RailsCast](http://railscasts.com/episodes/270-authentication-in-rails-3-1) explores it as the second option (It hasn't changed much since Rails 3.1 but keep in mind that some of the other Rails code is outdated, such as `validates_presence_of` or `find_by_email`).

Anyway, let's create our User model:

    rails g model User email password_digest

As before, this will generate a new migration along with the user.rb file. Run the `rake db:migrate` command next to have the changes take effect.

In order to support digested passwords, we need to activate (uncomment) the `bcrypt-ruby` gem in the Gemfile: `gem 'bcrypt-ruby', '~> 3.0.0'`. Remember to run the `bundle install` command right after saving this change since our dependencies (Gemfile) have changed!

Add the validations that you see in the commit diff. Note how a `password` field is validated. Two things you might notice here:

1. We didn't specify a `password` attribute in our generator/migration. We only specified a `password_digest` attribute
2. The validation only occurs on creation, not on updating the record.

Open up `models/user.rb` and add in `has_secure_password`. The `has_secure_password` method adds a "virtual" attribute to the model, one that does not get saved in the database. We don't want to store the user's plain text password in our db, because that would be a security issue. We only store the digested version.

This means that the password attribute is provided by the user upon creation only and we validate its length only in that case.

Oh wait, we've forgotten the name fields for the model. Let's add them as part of a new migration:

    rails g migration add_name_fields_to_users firstname lastname

Open the new migration file in your editor and notice how it added firstname and lastname to the model based on the two parameters we passed the generator. Nailed it!

## 8. generating users controller and users/new view [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/6e03bd23b98ad51b3379f185c56b2957a02d55f7)

Great, now we actually have to provide a place for users to "create" themselves: a sign up page.

You know the drill. Add this to `routes.rb`:

    resources :users, only: [:new, :create]

Now it's time to create the controller for users, type this into the console:

    rails g controller users new create

Then add the information to the controller below so it looks as expected:

    class UsersController < ApplicationController

      def new
        @user = User.new
      end

      def create
        @user = User.new(user_params)

        if @user.save
          redirect_to movies_path
        else
          render :new
        end
      end

      protected

      def user_params
        params.require(:user).permit(:email, :firstname, :lastname, :password, :password_confirmation)
      end

    end

Since you've now read about `has_secure_password`, you know it adds virtual attributes `password` and `password_confirmation` to our user model. You can see them there in our `user_params` method, which means we'll use those as we build our form as well. `users/new.html` should look like this:

    <h1>Sign Up</h1>
    <%= form_for @user do |f| %>
      <% if @user.errors.any? %>
        <div>
          <%= pluralize(@user.errors.count, "error") %> prevented the account from being created:
          <ul>
            <% @user.errors.full_messages.each do |msg| %>
              <li><%= msg %></li>
            <% end %>
          </ul>
        </div>
      <% end %>
      <div>
        <%= f.label "email" %><br>
        <%= f.text_field "email" %>
      </div>
      <div>
        <%= f.label "firstname", "First Name" %><br>
        <%= f.text_field "firstname" %>
      </div>
      <div>
        <%= f.label "lastname", "Last Name" %><br>
        <%= f.text_field "lastname" %>
      </div>
      <div>
        <%= f.label "password" %><br>
        <%= f.password_field "password" %>
      </div>
      <div>
        <%= f.label "password_confirmation" %><br>
        <%= f.password_field "password_confirmation" %>
      </div>
      <div><%= f.submit "Submit" %></div>
    <% end %>

Notice we've again included code to display our validation errors if, for example, the user's password is too short or doesn't match.

## 9. adding log in functionality [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/b2e04a48647b007d8d76747a88d983f41c508772)

In the spirit of REST, while setting up our log in (and eventually, log out) functionality, it's convention to have a `session` resource that can be "created" and "destroyed." If we "create" a session, we're logging in. If we "destroy" one, we're logging out. With that in mind, let's look at what we'll add to `routes.rb` first:

    resources :sessions, only: [:new, :create]

Let's `rails g controller sessions new create` and make our `SessionsController` look like this:

    class SessionsController < ApplicationController

      def new
      end

      def create
        user = User.find_by(email: params[:email])

        if user && user.authenticate(params[:password])
          session[:user_id] = user.id
          redirect_to movies_path
        else
          render :new
        end
      end

    end

Already, things are a bit different here. This is largely due to the fact that we're not actually dealing with a model. (It's important to note that "resources" usually refer to models but not always.)

Because this controller isn't responsible for CRUD for a model, you'll notice we haven't instantiated anything in our `new` action. And our `create` action looks quite a bit different than our typical `create`:

First, we find the user based on the email in the "email" input field. Then, we make sure that user exists and can be authenticated by the password in the "password" input field. If so, we set the `:user_id` key in the `sessions` hash to the user's id. As you can guess, this is how we'll soon determine if a user is logged in.

Now that we know how we're going to handle the tracking of logged in users, let's auto-login users upon signup from our `UsersController`:

    class UsersController < ApplicationController

      [...]

      def create
        @user = User.new(user_params)

        if @user.save
          session[:user_id] = @user.id # auto log in
          redirect_to movies_path
        else
          render :new
        end
      end

      [...]

    end

Finally, let's take care of our form to complete this functionality. `sessions/new.html.erb` should look like this:

    <h1>Log In</h1>
    <%= form_tag sessions_path do %>
      <div>
        <%= label_tag "email" %><br/>
        <%= text_field_tag "email" %>
      </div>
      <div>
        <%= label_tag :password %><br/>
        <%= password_field_tag :password %>
      </div>
      <div><%= submit_tag "Log In" %></div>
    <% end %>

For the same reason we didn't instantiate an object in our `users#new` action, we haven't used a `form_for` here. It's a simple `form_tag` which will submit a POST request to the `sessions_path`, triggering our `sessions#create` action. Excellent.

## 10. displaying flash notices and alerts [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/8d80c1ba6d7eafca989c5602612949e6ff81d806)

Have you ever noticed those little notices that appear (usually just once) at the top of the page to let you know you've successfully accomplished something: logging in, signing up, submitting a form, requested a new password, whatever. In Rails, we store these little bits of text in the [flash hash](http://api.rubyonrails.org/classes/ActionDispatch/Flash/FlashHash.html).

Since these messages could pop up anywhere in our application, let's head to `app/views/layouts/application.html.erb` and edit so that it looks like the following:

    <!DOCTYPE html>
    <html>
    <head>
      <title>RottenMangoes</title>
      <%= stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true %>
      <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
      <%= csrf_meta_tags %>
    </head>
    <body>
      <% flash.each do |key, value| %>
        <%= content_tag(:div, value) %>
      <% end %>
      <%= yield %>
    </body>
    </html>

We've added an iterator through each of the flash keys and are displaying their values. Looks great, but we need to actually _set_ these key-value pairs! Let's add some code to our controllers thus far:

    class MoviesController < ApplicationController

      [...]

      def create
        @movie = Movie.new(movie_params)

        if @movie.save
          redirect_to movies_path, notice: "#{@movie.title} was submitted successfully!"
        else
          render :new
        end
      end

      [...]

    end

***

    class SessionsController < ApplicationController

      [...]

      def create
        user = User.find_by(email: params[:email])

        if user && user.authenticate(params[:password])
          session[:user_id] = user.id
          redirect_to movies_path, notice: "Welcome back, #{user.firstname}!"
        else
          flash.now[:alert] = "Log in failed..."
          render :new
        end
      end

    end

***

    class UsersController < ApplicationController

      [...]

      def create
        @user = User.new(user_params)

        if @user.save
          session[:user_id] = @user.id
          redirect_to movies_path, notice: "Welcome aboard, #{@user.firstname}!"
        else
          render :new
        end
      end

      [...]

    end

Nice! Note there are two different ways we're setting the flash object.

1. As an option in the `redirect_to` methods. That `notice:` is really accomplishing `flash.now[:notice] =`
2. The old-fashioned way. `flash.now[:alert]`

Now when we successfully submit a form, we get feedback that we've done so. Seriously! Try it!

## 11. adding links to log in, sign up, or log out [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/9d85fc327171bf0ac009858376e9555adb5f5bc6)

Now that we've set up all this functionality, let's make some links to allow users to sign up, log in, or log out. But first, we need an easy, application-wide way to check if a user is logged in.

Naturally, we'll handle this in `app/controllers/application_controller.rb`:

    class ApplicationController < ActionController::Base
      # Prevent CSRF attacks by raising an exception.
      # For APIs, you may want to use :null_session instead.
      protect_from_forgery with: :exception

      protected

      def current_user
        @current_user ||= User.find(session[:user_id]) if session[:user_id]
      end

      helper_method :current_user
      
    end

As you can see, this helper method will return the logged in user by checking the `session[:user_id]` value we set upon log in. Now we can call `current_user` from both our controllers and views to check our log in status. Very useful!

Let's use that info right away to create conditional "log in," "sign up," and "log out" links in the page footer. If a user isn't logged in, they should see the "log in" and "sign up" links. If they are, they should see that they're logged in and a "log out" link.

From the console:

    touch app/views/layouts/_footer.html.erb

Like flashes, we want this footer to appear on each page. Making a `_footer.html.erb` in our global "/layouts" folder makes sense. It should look something like this:

    <footer>
      <small>
        <% if current_user %>
          Signed in as <%= current_user.email %> (<%= link_to "Log out", session_path("current"), method: :delete %>)
        <% else %>
          <%= link_to "Log In", new_session_path %> | <%= link_to "Sign Up", new_user_path %>
        <% end %>
      </small>
    </footer>

Awesome. Now let's render that from `app/views/layouts/application.html.erb` like so:

    <!DOCTYPE html>
    <html>
    <head>
      <title>RottenMangoes</title>
      <%= stylesheet_link_tag    "application", media: "all", "data-turbolinks-track" => true %>
      <%= javascript_include_tag "application", "data-turbolinks-track" => true %>
      <%= csrf_meta_tags %>
    </head>
    <body>
      <% flash.each do |key, value| %>
        <%= content_tag(:div, value) %>
      <% end %>
      <%= yield %>
      <%= render 'layouts/footer' %>
    </body>
    </html>

Perfect. Now we need to add the `sessions#destroy` endpoint we're trying to hit in our "log out" link above. Let's first add the route in `routes.rb`:

    resources :sessions, only: [:new, :create, :destroy]

Then the controller action in `SessionsController`:

    def destroy
      session[:user_id] = nil
      redirect_to movies_path, notice: "Adios!"
    end

Here we're simply clearing our `session[:user_id]` and redirecting. No sweat.

## 12. generating review model [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/05cdeefdaa079f0f680dda8a4bf1438704298742)

Now we're going to implement reviews. If logged in, users should be able to see movie reviews from other users and create their own. The first step here is to create our model:

    rails g model review user:references movie:references text:text rating_out_of_ten:integer

Okay `rake db:migrate` that thing.

You're probably wondering what kind of data type "references" is. Well, if you head to `db/migrate/[time_stamp]_create_reviews.rb`, you'll see `index: true` next to those two columns. And if you check out `schema.rb`, you'll see something slightly different:

    ActiveRecord::Schema.define(version: 20131117211410) do
      
      [...]
      
      create_table "reviews", force: true do |t|
        t.integer  "user_id"
        t.integer  "movie_id"
        t.text     "text"
        t.integer  "rating_out_of_ten"
        t.datetime "created_at"
        t.datetime "updated_at"
      end

      add_index "reviews", ["movie_id"], name: "index_reviews_on_movie_id"
      add_index "reviews", ["user_id"], name: "index_reviews_on_user_id"

      [...]

    end

This means in our reviews table, we'll have columns of `user_id` and `movie_id` indicating "this review belongs to this movie and this user." If you check out `app/models/review.rb`, you'll see Rails has in fact added these ActiveRecord methods for us:

    class Review < ActiveRecord::Base
      belongs_to :user
      belongs_to :movie
    end

This means we can call `@review.movie` or `@review.user` and access the movie or user the review belongs to. Most excellent.

Next, to allow us to call `@user.reviews` to access all reviews that belong to a given user or `@movie.reviews` to access all reviews for a give movie, we need to add those methods in `user.rb` and `movie.rb` as follows:

    class User < ActiveRecord::Base

      has_many :reviews

      [...]

    end

***

    class Movie < ActiveRecord::Base

      has_many :reviews

      [...]

    end

Awesome. Let's go ahead and add some validations to reviews. `review.rb` should look like this:

    class Review < ActiveRecord::Base

      belongs_to :user
      belongs_to :movie

      validates :user,
        presence: true

      validates :movie,
        presence: true

      validates :text,
        presence: true

      validates :rating_out_of_ten,
        numericality: { only_integer: true }
      validates :rating_out_of_ten,
        numericality: { greater_than_or_equal_to: 1 }
      validates :rating_out_of_ten,
        numericality: { less_than_or_equal_to: 10 }
        
    end

## 13. generating reviews controller and reviews/new view [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/782e415f83aabf712e757d21408320aae4b6ac8c)

Huzzah! Let's move on to actually letting logged in users see and create reviews. That's going to involve some routing first:

    RottenMangoes::Application.routes.draw do
      
      resources :movies do
        resources :reviews, only: [:new, :create]
      end
      resources :users, only: [:new, :create]
      resources :sessions, only: [:new, :create, :destroy]
  
    end

As you might have guessed, this is referred to as a [nested resource](http://guides.rubyonrails.org/routing.html#nested-resources). If you `rake routes` at this point, you'll see these lines added:

              Prefix Verb   URI Pattern                             Controller#Action
       movie_reviews POST   /movies/:movie_id/reviews(.:format)     reviews#create
    new_movie_review GET    /movies/:movie_id/reviews/new(.:format) reviews#new

This implies that whenever we're routed to the `ReviewsController`, each action will be in the context of the movie it belongs to. (We'll have access to `params[:movie_id]` to figure out which movie that is.)

Make sure to create the reviews controller with new and create. Then let's take a look at that controller and add to it:

    class ReviewsController < ApplicationController

      def new
        @movie = Movie.find(params[:movie_id])
        @review = @movie.reviews.build
      end

      def create
        @movie = Movie.find(params[:movie_id])
        @review = @movie.reviews.build(review_params)
        @review.user_id = current_user.id

        if @review.save
          redirect_to @movie, notice: "Review created successfully"
        else
          render :new
        end
      end

      protected

      def review_params
        params.require(:review).permit(:text, :rating_out_of_ten)
      end

    end

See how we're grabbing the correct movie by using `params[:movie_id]` at the beginning of each of these actions? Then we can automatically assign the `movie_id` to our new `@review` by building it like this:

    @review = @movie.reviews.build

The line above functions the same as this:

    @review = Review.new(movie_id: @movie.id)

Neato.

Let's take care of our `reviews/new.html.erb`:

    <%= link_to "Back to movie", movie_path(@movie) %><br/>

    <h2>Write a review of <em><%= @movie.title %></em>.</h2>

    <%= form_for([@movie, @review]) do |f| %>
      <% if @review.errors.any? %>
        <div>
          <%= pluralize(@review.errors.count, "error") %> prevented the review from being saved:
          <ul>
            <% @review.errors.full_messages.each do |msg| %>
              <li><%= msg %></li>
            <% end %>
          </ul>
        </div>
      <% end %>

      <div>
        <%= f.label "text" %><br>
        <%= f.text_area "text" %>
      </div>
      <div>
        <%= f.label "rating_out_of_ten" %><br>
        <%= f.number_field "rating_out_of_ten" %>
      </div>
      <div><%= f.submit "Submit" %></div>
    <% end %>

Pay close attention to what we're passing to `form_for`! This array syntax is how we handle nested resource forms in Rails.

Almost done, but we haven't actually displayed our reviews anywhere, have we? Instead of a `reviews#index` controller, we'll simply load our reviews (using our association) on `movies/show.html.erb`. It should now look something like this:

    <%= link_to "Back to all movies", movies_path %><br/>

    <%= link_to image_tag(@movie.poster_image_url), movie_path(@movie) %>
    <h2><%= @movie.title %> (<%= link_to "edit", edit_movie_path(@movie) %>, <%= link_to "delete", movie_path(@movie), method: :delete, confirm: "You sure?" %>)</h2>
    <h3><%= @movie.release_date %></h3>
    <h4>Dir. <%= @movie.director %> | <%= @movie.runtime_in_minutes %> minutes</h4>
    <p><%= @movie.description %></p>

    <hr>

    <h3>Reviews of <em><%= @movie.title %></em></h3>
    <% if current_user %>
      <% @movie.reviews.each do |review| %>
        <p><%= review.text %></p>
        <p><%= review.rating_out_of_ten %>/10</p>
        <small>- <%= review.user.email %></small><br/>
      <% end %>
      <p><%= link_to "Write a review!", new_movie_review_path(@movie) %></p>
    <% else %>
      <p>Please <%= link_to "log in", new_session_path %> to see reviews and add your own.</p> 
    <% end %>

Notice we've wrapped the good stuff in a check of `current_user` to make sure we're logged in.

This thing's really coming together!

## 14. using a before_filter in the reviews_controller [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/4759eb53083e46c21baf2812a236d3e369c466d8)

We're now in the "little clean up commits" portion of the tutorial. Welcome!

First up: refactoring our `ReviewsController` to utilize a [filter](http://guides.rubyonrails.org/action_controller_overview.html#filters). Filters are simply methods that run before, after, or "around" each controller action. In our `ReviewsController`, we have the same line at the beginning of both actions:

    @movie = Movie.find(params[:movie_id])

Let's refactor so that the controller now looks like this:

    class ReviewsController < ApplicationController

      before_filter :load_movie

      def new
        @review = @movie.reviews.build
      end

      def create
        @review = @movie.reviews.build(review_params)
        @review.user_id = current_user.id

        if @review.save
          redirect_to @movie, notice: "Review created successfully"
        else
          render :new
        end
      end

      protected

      def load_movie
        @movie = Movie.find(params[:movie_id])
      end

      def review_params
        params.require(:review).permit(:text, :rating_out_of_ten)
      end

    end

This is an very common use case: loading the parent resource from a nested resource controller. Get used to it!

## 15. restricting access to create reviews if !current_user [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/e49e1c66d37c8070acaef7fd699dfd89c3b9be08)

If you look at `movies/show.html.erb`, we've successfully hidden the reviews and the link to create one if the user isn't currently logged in. But what if they visit [localhost:3000/movies/1/reviews/new](http://localhost:3000/movies/1/reviews/new) directly? Uh oh. Let's implement another very common filter, this time in `app/controllers/application_controller.rb`, so we can access the method from anywhere:

    class ApplicationController < ActionController::Base
      # Prevent CSRF attacks by raising an exception.
      # For APIs, you may want to use :null_session instead.
      protect_from_forgery with: :exception

      protected

      def restrict_access
        if !current_user
          flash[:alert] = "You must log in."
          redirect_to new_session_path
        end
      end

      def current_user
        @current_user ||= User.find(session[:user_id]) if session[:user_id]
      end

      helper_method :current_user
      
    end

This is a dead simple method that will allow us to redirect and inform the user they must log in anytime they attempt to access a controller action we don't want them to.

We can implement it in our `ReviewsController` like so:

    class ReviewsController < ApplicationController

      before_filter :restrict_access
      
      [...]

    end

Boom! You ain't creating a review, Mr. Not Logged In Guy. Nice try.

## 16. using some helper methods [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/64930062bfbc6bbf306bc6ed6077e981a209ccc0)

Okay it's time to finally add the main feature of Rotten Mangoes: the average review! We could theoretically calculate this anywhere: the view inside ERB or the controller method where we want to display it. But methods like these, "business logic," belongs in the appropriate model. So let's head to `movie.rb` and add a method to average the reviews.

    class Movie < ActiveRecord::Base

      [...]

      def review_average
        reviews.sum(:rating_out_of_ten)/reviews.size
      end

      protected

      [...]

    end

Caution: if a movie has no reviews, reviews.size will return 0. Figure out a way to handle this situation.

Notice this method isn't protected, so we'll be able to call it from anywhere within the app. Like, say, on our `movies/index.html.erb` and `movies/show.html.erb`? Just add this ERB wherever you want to display the average:

    <%= @movie.review_average %>/10

Okay awesome. Now let's tackle those really, really ugly `release_date`s we're displaying. It's also tempting to put this date cleaner-upper method in our movie model, but since it will really only be used in our views, we're going to use a helper method instead.

Did you notice Rails was creating these files every time you `rails g`'d a controller? Let's open up `app/helpers/movies_helper.rb` add this date formatter:

    module MoviesHelper
  
      def formatted_date(date)
        date.strftime("%b %d, %Y")
      end

    end

Now wherever we display our `release_date`, we can easily format it by replacing the old code with

    <%= formatted_date(@movie.release_date) %>

By the way, [this](http://www.foragoodstrftime.com/) is a useful (and wonderfully named) little web app to help with strftime formatting.

Okay our last bit of refactoring is writing a method to display a user's full name. This is another very common method in user models. Let's go for it in `users.rb`:

    class User < ActiveRecord::Base

      [...]

      def full_name
        "#{firstname} #{lastname}"
      end

    end

So instead of displaying a user's email in our views, let's display their full name. In `app/views/layouts/_footer.html.erb`:

    <footer>
      <small>
        <% if current_user %>
          Signed in as <%= current_user.full_name %> (<%= link_to "Log out", session_path("current"), method: :delete %>)
        <% else %>
          <%= link_to "Log In", new_session_path %> | <%= link_to "Sign Up", new_user_path %>
        <% end %>
      </small>
    </footer>

And on each review on `movies/show.html.erb`:

    [...]
    <% @movie.reviews.each do |review| %>
      <p><%= review.text %></p>
      <p><%= review.rating_out_of_ten %>/10</p>
      <small>- <%= review.user.full_name %></small><br/>
    <% end %>
    [...]

## 17. routing root to movies#index [★](https://github.com/lighthouse-labs/rotten_mangoes/commit/ad53e1e51d2c68908eea59f3e6941e33d15550e1)

Last but not least: let's route the root of our app ([localhost:3000](http://localhost:3000)) to `movies#index`! In `routes.rb`:

    RottenMangoes::Application.routes.draw do
  
      resources :movies do
        resources :reviews, only: [:new, :create]
      end
      resources :users, only: [:new, :create]
      resources :sessions, only: [:new, :create, :destroy]
      root to: 'movies#index'

    end

***

Congrats! We have a pretty legit app already! Users, sessions, nested resources. This is good stuff. If you're itching to push ahead, here are some ideas for "extra credit."

1. add Actor model and necessary associations and models (hint: this will involve using `has_many, through:`!)
2. add movie categories
3. sort movies by different properties (average review, title, newest, et cetera)
4. add users/show.html with list of movies reviewed and average review
5. stylin'! `bootstrap-sass` or `foundation` gem