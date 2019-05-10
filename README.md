# Custom JSON Rendering Using Rails

## Learning Goals

- Render JSON from a Rails controller
- Define the JSON content being sent

## Introduction

By using `render json:` in our Rails controller, we can take entire models or
even collections of models, have Rails convert them to JSON, and send them out
on request. We already have the makings of a basic API. In this lesson, we're
going to look at shaping that data that gets converted to JSON and make it more
useful to us from the frontend JavaScript perspective.

The way we structure our data matters - it can lead to better, simpler code in
the future. By specifically defining what data is being sent via a Rails
controller, we have full control over what data our frontend has access to.

To follow along, run `rails db:migrate` and `rails db:seed` to set up your
database and example data. We will continue to use our bird watching example in
this lesson.

## Adding Additional Routes to Separate JSON Data 

The simplest way to make data more useful to us is to provide more routes and
actions that help to divide and organize our data. For instance, we could add a
`show` action to allow us to send specific record/model instances. First, we'd
add a route:

```ruby
Rails.application.routes.draw do
  get '/birds' => 'birds#index'
  get '/birds/:id' => 'birds#show' # new
end
```

Then we could add an additional action:

```ruby
class BirdsController < ApplicationController
  def index
    @birds = Bird.all
    render json: @birds
  end

  def show
    @bird = Bird.find(params[:id])
    render json: @bird
  end
end
```

Now, visiting `http://localhost:3000/birds` will produce an array of Bird
objects, but `http://localhost:3000/birds/2` will produce just one:

```ruby
{
  "id": 2,
  "name": "Grackle",
  "species": "Quiscalus Quiscula",
  "created_at": "2019-05-09T21:51:41.543Z",
  "updated_at": "2019-05-09T21:51:41.543Z"
}
```

We can use multiple routes to differentiate between specific requests. In an
API, these are typically referred to as endpoints a user of the API could use to
access specific pieces of data. We could build a fully functional CRUD resource
rendering JSON and interact with it purely with JavaScript `fetch()` requests. 

> **ASIDE:** If you've ever tried using `rails generate scaffold` to create a
resource, you'll find that this is the case. Rails has favored convention over
configuration and will set up JSON rendering for you almost immediately out
of the box.

In terms of communicating with JavaScript, even when sending POST requests, we
do not need to change anything in our controller to handle a `fetch()` request
compared to a normal user visiting a page. This means that you could go back to
_any_ existing Rails project and all you would need to do is change the
rendering portion of the controller to make it render JSON. Bam! You have a
rudimentary Rails API!

Even though we are no longer serving up views the same way, maintaining RESTful
conventions is still a HUGE plus here for your end API user (mainly yourself at
the moment). We wouldn't want to pollute a Rails controller with a ton of extra
actions in order to further customize how our data is shaped. To shape
what data looks like further, we'll want to look at other options when rendering.

## Removing Content when Rendering

Sometimes, when sending JSON data, such as an entire model, we don't want or
need to send the entire thing. Some data is sensitive - for instance, an API that
sends user information might store details of a user that it uses internally, but
does not want ever send externally on request. Sometimes, data is just extra 
clutter we don't need. Consider the last piece of data:

```ruby
{
  "id": 2,
  "name": "Grackle",
  "species": "Quiscalus Quiscula",
  "created_at": "2019-05-09T21:51:41.543Z",
  "updated_at": "2019-05-09T21:51:41.543Z"
}
```

For our bird watching purposes, we probably don't need bits of data like
`created_at` and `updated_at`. Rather than send this unnecessary info when
rendering, we can cut it out. The simplest way would be to use Ruby's built-in
`slice` method. On the `show` action, that would look like this:

```ruby
def show
  @bird = Bird.find(params[:id])
  render json: @bird.slice{:id, :name, :species}
end
```

The `Hash` [`slice` method][slice] returns a hash with only the keys that are
passed into `slice`. In this case, `:id`, `:name`, and `:species` were passed,
in, so `created_at` and `updated_at` get cut out.

[slice]: https://ruby-doc.org/core-2.5.0/Hash.html#method-i-slice

The result is that when we visit a specific bird's endpoint, like
`http://localhost:3000/birds/3`, we'll see just the name and species:

```js
{
  "id": "3",
  "name": "Common Starling",
  "species": "Sturnus Vulgaris"
}
```

Cool, but once again, Rails has one better. While `slice` works fine for a
single hash, as with `@bird`, it won't work for an array of hashes like the one
we have in our `index` action:

```ruby
def index
  @birds = Bird.all
  render json: @birds
end
```

In this case, Rails provides the `only:` keyword that we can add directly
after listing an object we want to render to JSON.

```ruby
def index
  @birds = Bird.all
  render json: @birds, only: [:id, :name, :species]
end
```

Visiting or fetching `http://localhost:3000/birds` will now produce our array of
bird objects and each object will _only_ have the `id`, `name` and `species`
values, leaving out everything else:

```ruby
[
  {
    "id": 1,
    "name": "Black-Capped Chickadee",
    "species": "Poecile Atricapillus"
  },
  {
    "id": 2,
    "name": "Grackle",
    "species": "Quiscalus Quiscula"
  },
  {
    "id": 3,
    "name": "Common Starling",
    "species": "Sturnus Vulgaris"
  },
  {
    "id": 4,
    "name": "Mourning Dove",
    "species": "Zenaida Macroura"
  }
]
```

Alternatively, rather than specifically listing every key we want to include, we
could also exclude particular content using `except:`, like so:

```ruby
def index
  @birds = Bird.all
  render json: @birds, except: [:created_at, :updated_at]
end
```

The above code would achieve the same result, producing all `id`, `name`, and
`species`. All the keys _except_ `created_at` and `updated_at`.

## Conclusion

We can now take a single model or all the instances of that model and render it
to JSON, extracting out any specific content we do or do not want to send!

Whether you are building a professional API for a company or for your own
personal site, having the ability to fine tune how your data look is a critical
skill that we're only just beginning to scratch the surface on.

In the next lesson, we're going to continue to look at options for customizing
rendered JSON content. Particularly, we'll be looking more at what we can _add_.