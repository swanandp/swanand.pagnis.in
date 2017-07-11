---
layout: post
title: "Consuming HTTP APIs in Ruby"
date: 2017-07-08 18:07:41 +0530
comments: true
categories: ruby http-api rest-api consuming-api httparty
---

What is your favourite technique for consuming HTTP APIs in Ruby?  I like using [HTTParty][1]!

- It offers a simple, intuitive API.
- It makes it easy to support a whole bunch of standard API authentication mechanisms. <!-- more -->
- It de-serialises responses based on content type headers.
- It allows us to write simple wrappers that are very close in form to the API we want to communicate with.
- It has a nice name!  And we know [how hard it is to name things][2].

I have come to employ a few patterns when working with HTTParty.  They are all centered around having a convenient internal API to work with, and the ease of testing.

Most APIs I've worked with have one of the following authentication mechanism:

- HTTP Basic Authentication
- HTTP Digest Authentication
- Auth token in header
- API key in query params / request body


I am really fond of examples, so let's consider an example.  A RESTful service, where we interact with the "Post" resource.  We're able to list posts, get details about a post, create, update & delete a post.  The service demands HTTP Basic auth, and JSON encoding.

```ruby

class PostsService
  include HTTParty

  base_uri "https://api.example.com"
  read_timeout 5 # always have timeouts!
  # debug_output $stdout # for quick access during debugging

  attr_reader :auth, :headers

  def initialize
    @auth = {
      username: ENV["API_USERNAME"],
      password: ENV["API_PASSWORD"],
    }

    @headers = {
      "Content-Type" => "application/json",
    }
  end

  def index
    get("posts")
  end

  def show(post_id)
    get("posts/#{post_id}")
  end

  def create(attributes)
    self.class.post(
      endpoint("posts"),
      default_options.merge(body: attributes.to_json)
    )
  end

  def update(post_id, attributes)
    self.class.patch(
      endpoint("posts/#{post_id}"),
      default_options.merge(body: attributes.to_json)
    )
  end

  def destroy(post_id)
    self.class.delete(
      endpoint("posts/#{post_id}"),
      default_options
    )
  end

  protected

  def default_options
    {
      headers: headers,
      basic_auth: auth,
    }
  end

  def endpoint(uri)
    "/v1/#{uri}"
  end

  def get(uri)
    self.class.get(
      endpoint(uri),
      default_options
    )
  end
end

```

Here's why I like this code:

1. Simple, easy to read code that mimics the API quite nicely.
2. Intuitive. Sending a POST request, is as simple as calling `post`.  No need to remember multiple things. Specifying headers, is literally passing an argument called `headers`.
4. Interacting with the API is now simple:

```ruby

service = PostsService.new

pp service.index
# []

# Create a post
response = service.create(
  name: "Star Trek: A new hope",
  body: "A play about how Frodo is
         tricked into attending the
         tri-wizard tournament by
         evil shogun Gandalf"
)

# Made a mistake in the title, update it
service.update(
  response['post']['id'],
  name: "Star Wars: Into the darkness"
)

# Delete that abomination of a post
service.delete(response['post']['id'])

```

Where's the fun without ever changing requirements?

API, now V2, demands that we use digest auth, instead of basic auth.

```ruby
def default_options
  {
    headers: headers,
    digest_auth: auth,
  }
end

def endpoint(uri)
  "/v2/#{uri}"
end
```

That was a simple change. Let's try adding custom headers.  The API now supports logging, tracking and tagging requests. All done through headers. Since this is context specific, we'll pass in the context as an argument to the constructor:

```ruby
def initialize(user)
  @auth = {
    username: ENV["API_USERNAME"],
    password: ENV["API_PASSWORD"],
  }

  @headers = {
    "Content-Type" => "application/json",
    "X-User-ID" => user.tracking_id,
    "X-Tags" => "Name:#{user.full_name}",
  }
end
```

In addition to basic auth,  we should now send in an oauth style "Bearer" token:

```ruby
@headers = {
  "Content-Type" => "application/json",
  "X-User-ID" => user.tracking_id,
  "X-Tags" => "Name:#{user.full_name}",
  "Authorisation" => "Bearer:#{user.oauth_token}",
}
```

Okay, this last example was lame.  But the point remains, HTTParty allows you to build your own service objects, and gets out of your way.  Exactly what a library should do.

If you really take a close look at our examples, consuming an HTTP API is all about:

- Setting Headers
- Specifying Query parameters
- Request body encoding
- Parsing response based on content type
- Making the HTTP requests


Not surprising, right?  HTTParty makes it simple, and intuitive to perform all these actions.  And hence it remains my favourite.


P.S.  Here's the code used above, [in a Gist][4].


<!-- Links -->

[1]: https://github.com/jnunemaker/httparty
[2]: https://twitter.com/timbray/status/817025379109990402?cn=cmVwbHk%3D
[3]: https://developers.google.com/places/web-service/autocomplete
[4]: https://gist.github.com/swanandp/51d24ca474b10b10c68f2afeb30dc65e
