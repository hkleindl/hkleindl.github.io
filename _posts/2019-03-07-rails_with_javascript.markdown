---
layout: post
title:      "Rails with Javascript"
date:       2019-03-07 13:23:02 -0500
permalink:  rails_with_javascript
---


Javascript is the language that introduced me to programming. It taught me many of the fundamental programming concepts such as data types, loops, and conditional statements. I didn't have much of an idea of its practical use, but it built a foundation for future learning. 
	
I took a break from Javascript to learn how to make basic web applications with Ruby on Rails, HTML, and CSS. In the back of my mind, I kept wondering where Javascript fit in. I could already build dynamic web pages, so how much more was there to learn? As it turned out, a lot. 
	
When I returned to Javascript, I started to value its versatility in making great sites. Not only could it change DOM elements on the fly with event listeners, but it could also make AJAX requests without reloading the page. 
	
AJAX is the real game changer. It makes getting information from the server possible without page loads for a more seamless user experience. For example, in this simplified version of MyBookShelf, I have a book Model:
	
```
class Book < ApplicationRecord
	belongs_to: author
end
```

In the book controller:

```
def index
	@books = Book.all
end

def show
	@book = Book.find(id: params[:id])
end
```

And the book index view:

```
<% @books.each do |b|>
	<%= b.title>
<% end>
```

The book index page is just a simple list of each book's title. I could render more information about every single book, but that would clutter up the list. The book's show page could display more information, but that requires waiting for a page reload. What if I could render additional information about just the book we want without reloading the page? That's where AJAX comes in.

In the index view, I'll add a button element with a data attribute of `book-id` that has the value of the book's id. I've also added an empty div element with an `id` attribute that contains the book's id as well:

```
<% @books.each do |book|>
    <%= book.title>
    <button data-book-id="<%= book.id %>">details</button>
    <div id="book-<%= book.id %>-details"></div>	
<% end>
```

Now I need to update my controller to handle JSON responses as well as HTML: 

```
def show
    @book = Book.find(id: params[:id])
    respond_to do |f|
              f.html 
              f.json { render json: @book, status: 200 }
    end
end
```

This way, navigating to the book's show page will render HTML as normal,  but we can also send a request to receive a JSON representation of the book object. To do that, I'll use Rails' Active Model Serializer to format the book object to JSON:

```
class BookSerializer < ActiveModel::Serializer
  attributes :id, :title, :author_name
end
```

This will give us a nice JSON representation of the book that we can manipulate with Javascript:

```
{
	id: 1,
	title: "Slaughterhouse Five",
	author_name: "Kurt Vonnegut"
}
```

Now, for the Javascript:

```
$(function() {
  $('button').on('click', function() {
    let bookId = $(this).data('book-id')
    $.getJSON(`/books/${bookId}`, function(response) 
      $(`#book-${response.id}-details`).html(response.author_name).toggle()
    })
  })
})
```

Here I'm using Jquery to select the button element and adding a click event listener. When the button is clicked,  `bookId` is assigned the value of 'this' element's `data-book-id` attribute. This value matches the id of the book we want to query.

Then I use the `getJSON` method which sends an AJAX call to retrieve the book's JSON representation. The request is sent via the passed in URL of  `/books/${bookId}`, and in the callback function is where I can access the response that contains the response JSON. 

Inside of the callback I set the HTML content of our previously empty div element to the `author_name` provided by the response JSON. I add the `toggle` method on the end which will hide or show the div content with each click. Now every time I click the button a request is sent to the server and returned with more data. All *without* reloading the page. Cool!

Javascript is a very powerful tool that can add a lot of interactivity to a site for richer user experiences. I have a greater appreciation of its functionality now that I have experience with using it in the context of an application.  
