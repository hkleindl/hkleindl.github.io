---
layout: post
title:      "Sinatra Workout Logger"
date:       2018-01-18 17:57:31 +0000
permalink:  sinatra_workout_logger
---


I've really enjoyed working with Sinatra and experiencing its ability to bridge the gap between the back-end and front-end of an application. It has given everything I've learned about Ruby/ActiveRecord a fresh sense of purpose. As a result, I was eager to get started on my own Sinatra project.

#### Models

I wanted to create a simple workout logger application to keep track of a user's exercises. I had a relatively clear picture of how I wanted to set everything up. I knew that I wanted `User`, `Workout`, and `Exercise` models, but figuring out how I wanted to associate them proved to be a little tricky. My first idea was something like:

```
class User < ActiveRecord::Base
   has_many :workouts
   has_many :exercises, through: :workouts
end

class Workout < ActiveRecord::Base
   belongs_to :user
   has_many :exercises
end

class Exercise< ActiveRecord::Base
   belongs_to :workouts
   has_many :users, through: :workouts
end
```

One of the problems with these associations is that an `Exercise` object can belong to more than one `User`. I wanted each exercise to be unique to the user that created it. After playing around a bit, I settled on:

```
class User < ActiveRecord::Base
  has_many :workouts
  has_many :exercises
end

class Exercise < ActiveRecord::Base
  belongs_to :user
  belongs_to :workout
end


class Workout < ActiveRecord::Base
  belongs_to :user
  has_many :exercises
end
```

I'm still not sure if this is the best way to relate the models, but it works so I'll take it! 

#### Tables

Next, I created tables for each of my models.  

```
class CreateUsers < ActiveRecord::Migration[5.1]
  def change
    create_table :users do |t|
      t.string :email
      t.string :username
      t.string :password_digest
    end
  end
end

class CreateWorkouts < ActiveRecord::Migration[5.1]
  def change
    create_table :workouts do |t|
      t.string :date
      t.text :notes
      t.integer :user_id
    end
  end
end

class CreateExercises < ActiveRecord::Migration[5.1]
  def change
    create_table :exercises do |t|
      t.string :category
      t.string :name
      t.string :description
      t.integer :weight
      t.integer :reps
      t.integer :sets
      t.string :intensity
      t.integer :minutes
      t.integer :workout_id
      t.integer :user_id
    end
  end
end
```

In the `exercises` table, `category` would represent either a resistance exercise or a cardio exercise. All `Exercise` objects would be created with a `name`, optional `description`, `workout_id`, and `user_id`. If the `category` of an `Exercise` object was created with a value of  `'resistance'`, then the object would have values for `weight`, `reps`, and `sets`. If `category` had a value of `'cardio'`, the object would be created with values for `intensity` and `minutes`. It seemed a bit sloppy to be creating objects with a bunch of nil values. I considered the idea of having separate `Cardio` and `Resistance` models that inherited from `Exercise`, but that would've added an extra layer of complexity I wasn't sure how to handle. 

#### Controllers

I made a `UsersController`, `WorkoutsContoller`, and `ExercisesController`, each of which inherited from an `ApplicationController`. `ApplicationController` held the route for the home page and a few helper methods to use in the other controllers. I also added `use Rack::Flash` to display messages based on undesired form input. 

`UsersController` had routes for signing up, logging in, and logging out. I made sure to account for username and email duplicates, making sure that a user could only signup using a username and email that did not match another user's in the database. The logging in actions searched the database for a `User` that had a matching `username` entered in the login form, then used the Bcrypt method, `authenticate`, to check password validation. If those conditions were satisfied, the corresponding `User`'s id was stored in `session[:user_id]`. The logging out action simply destroyed `session[:user_id]`.

`WorkoutsController` was responsible for the actions to create, view, edit, and delete `Workout`s. Every time a workout was created, it was associated with the current user. 

`ExercisesController` held the CRUD actions for `Exercise`s. Since I didn't have designated `Cardio` or `Resistance` models, I needed to have unique routes that would display different forms based on the value of `Exercise.category`. After creating a new `Exercise`, the new object was added to `User.exercises`. Additionally, I made routes to associate an exercise with one of the current user's workouts, delete an exercise from a workout, and delete an exercise from the user's exercises.

#### Views

I encountered some interesting problems with a few of my ERB views. I wanted to have one view for creating an exercise. I was hoping I could use an `if` statement to display certain fields of a form. I was thinking something like:

```
<input type="radio" value="cardio">Cardio</input>
<input type="radio" value="resistance">Resistance</input>

# If 'cardio' is selected
#    display 'intensity' and 'minutes' fields
# elsif 'resistance' is selected
#     display 'sets', 'reps', and 'weight' fields
# end  
```

Googling around led me to the conclusion that I wouldn't be able to do this without using Javascript. I decided that adding another language was probably not going to be worth it. Instead, I just made specific routes and views for creating an `Exercise` with corresponding attributes based on what the `category` was.

I had a similar issue with the view that displayed a workout and all its exercises. My strategy was to iterate through all the exercises that belonged to that workout, then display each attribute of the exercise. The problem was that an exercise with `category: 'cardio'` would have nil values for `weight`, `sets` and `reps` and an exercise with `category: 'resistance'` would have nil values for `intensity` and `minutes`. To fix this I added an `if` statement, checking for the value of `category` and displaying only certain attributes based on that value. 

#### Conclusion

I'm mostly satisfied with how the application turned out. There were a few issues I approached in a somewhat hacky manner but I feel confident that as I continue to learn, I will acquire additional tools to address these issues more elegantly. 










