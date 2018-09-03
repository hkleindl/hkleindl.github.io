---
layout: post
title:      "Rails MyBookShelf Project"
date:       2018-09-03 19:27:22 +0000
permalink:  rails_mybookshelf_project
---


For my Ruby on Rails project, I built an application called MyBookShelf that allows users to keep track of and rate books they have read. This is the first Rails app that I have built from scratch which gave me a chance to delve deeper into the framework and get to know it a bit better. I also encountered some interesting challenges with validations along the way. 

One of the first snags I ran into while working on my app was when I attempted to instantiate a new Book object and add an existing User object to the book's list of users. After filling out and submitting the form on the '/users/:id/books/new' page, the `create`  action in my Book controller redirects to the '/users/:id' page that contains a list of all the books the user has added. The problem was that the newly created Book object wasn't showing up on the page. I dropped into a console session to see if I could figure out where the bug was coming from.

In the console, I ran the logic in the `create` action, line by line, to get a better idea of where my code was breaking. First I instantiated a new Book:

``` ruby
2.3.6 :002 > @book = Book.new(name: "Moby Dick", author_name: "Herman Melville")
  Author Load (0.4ms)  SELECT  "authors".* FROM "authors" WHERE "authors"."name" = ? LIMIT ?  [["name", "Herman Melville"], ["LIMIT", 1]]
   (6.6ms)  begin transaction
  Author Create (5.1ms)  INSERT INTO "authors" ("name", "created_at", "updated_at") VALUES (?, ?, ?)  [["name", "Herman Melville"], ["created_at", "2018-09-01 15:07:41.589799"], ["updated_at", "2018-09-01 15:07:41.589799"]]
   (16.0ms)  commit transaction
 => #<Book id: nil, name: "Moby Dick", created_at: nil, updated_at: nil, author_id: 12>  
```  

So far so good.  Next, I added an existing User to the new Book's users:

```ruby
2.3.6 :003 > @book.users << User.last
  User Load (2.9ms)  SELECT  "users".* FROM "users" ORDER BY "users"."id" DESC LIMIT ?  [["LIMIT", 1]]
 => #<ActiveRecord::Associations::CollectionProxy [#<User id: 9, username: "bruce", password_digest: "$2a$10$Fd.wiE5mMvbpiai2Upj00eSJCWFHRuIYMQnavtrRiKg...", created_at: "2018-08-21 17:11:35", updated_at: "2018-08-21 17:11:35", email: "bruce@email.com">]>
 ```
 
Then, when I tried saving the book is when things broke:

```ruby
2.3.6 :004 > @book.save
   (0.1ms)  begin transaction
  User Exists (0.2ms)  SELECT  1 AS one FROM "users" WHERE "users"."username" = ? AND "users"."id" != ? LIMIT ?  [["username", "bruce"], ["id", 9], ["LIMIT", 1]]
  User Exists (0.2ms)  SELECT  1 AS one FROM "users" WHERE "users"."email" = ? AND "users"."id" != ? LIMIT ?  [["email", "bruce@email.com"], ["id", 9], ["LIMIT", 1]]
  Book Exists (7.7ms)  SELECT  1 AS one FROM "books" WHERE "books"."name" = ? AND "books"."author_id" = ? LIMIT ?  [["name", "Moby Dick"], ["author_id", 12], ["LIMIT", 1]]
   (0.1ms)  rollback transaction
 => false
```

Okay, so why wouldn't the Book save? I checked the errors:

```ruby
2.3.6 :006 > @book.errors.full_messages
 => ["Users is invalid"]
```

I was pretty confused at this point since I shouldn't have been able to create an invalid User in the first place. I checked the User to see what the errors were:

```ruby
2.3.6 :019 > @book.users.first.errors.full_messages
 => ["Password can't be blank", "Password is too short (minimum is 6 characters)"]
```

This had me stumped for a while. In my User model I had already included a validation to make sure there is a password value when a User is created:

```ruby
class User < ApplicationRecord

   has_secure_password

   validates :password, presence: true, length: { in: 6..20 }

end
```

I didn't understand how I could have originally created a User with a password and then have it later fail the validations. Then I realized that because I was using `has_secure_password`, the password attribute isn't stored:

```ruby
2.3.6 :020 > @book.users.last.password
 => nil
 ```
 
Initially, when the form is filled out to create a User, there must be a password attribute. But after the creation of the User, `has_secure_password` stores an encrypted `password_digest` hash and the password becomes `nil`. Since I put `validates :password, presence: true` in the User model, there was a conflict with validations `has_secure_password` automatically adds. The [source code](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/secure_password.rb "rails/secure_password.rb") states:

```
# The following validations are added automatically:
# * Password must be present on creation
# * Password length should be less than or equal to 72 bytes
# * Confirmation of password (using a +XXX_confirmation+ attribute)
```
	
 Not only was adding my own validation for password presence redundant but `has_secure_password` validates for password presence _on creation_, which allows the password to be set to `nil` for safety _after_ the creation of the User object. The validation I had originally included persisted even after the User was created.
 
The solution I found was to remove my password presence validation and to add:
	
```ruby
class User < ApplicationRecord
  
   has_secure_password
 
   validates :password, length: { minimum: 6 }, on: :create
                      
end
  
```	  
	
I kept the validation for password length, but needed to add `on: :create` so that the validation would only run on creation of the User. 

I went back into my console and tried to save the Book again:

```ruby
2.3.6 :001 > @book = Book.new(name: "Moby Dick", author_name: "Herman Melville")
=> #<Book id: nil, name: "Moby Dick", created_at: nil, updated_at: nil, author_id: 12>
 
2.3.6 :002 > @book.users << User.last
 => #<ActiveRecord::Associations::CollectionProxy [#<User id: 9, username: "bruce", password_digest:   "$2a$10$Fd.wiE5mMvbpiai2Upj00eSJCWFHRuIYMQnavtrRiKg...", created_at: "2018-08-21 17:11:35", updated_at: "2018-08-21 17:11:35", email: "bruce@email.com">]>
 
2.3.6 :004 > @book.save
 => true
```

That did the trick! Everything was now working as expected and the new book was showing up on the page. 

