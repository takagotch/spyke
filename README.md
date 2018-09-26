### spyke
---

https://github.com/balvig/spyke


```ruby
gem 'spyke'
{ data:{ id: 1, name: 'Bob' }, metadata: {}, errors: {} }
{ "result": { "id": 1, "name": "Bob" }, "extra": {}, "errors": {} }

vi config/initializers/spyke.rb
class JSONParser < Faraday::Response::Middleware
  def parse(body)
    json = MultiJson.load(body, symbolize_keys: true)
    {
      data: json[:result],
      metadata: json[:extra],
      errors: json[:errors]
    }
  end
end
Spyke::Base.connection = Faraday.new(url: 'http://api.com') do |c|
  c.request :json
  c.use JSONParser
  c.adapter Faraday.default_adapter
end


class User < Spyke::Base
  has_many :post
  scope :active, -> { where(active: true) }
end

User.all
# => GET http://api.com/users
User.active
# => GET http://api.com/users?active=true
user = where(age: 3).active
# => GET http://api.com/users?active=true&age=3
user = User.find(3)
# => GET http://api.com/users/3
user.posts
# => find embedded in returned data or GET http://api.com/users/e/posts
user.update(name: 'Alice')
# => PUT http://api.com/users/3 - { user: { name: 'Alice' } }
user.destroy
# => DELETE http://api.com/users/3
User.create(name: 'Bob')
# => POST http://api.com/users - { user: { name: 'Bob' } }


class User < Spyke::Base
  uri 'people(/:id)'
  
  has_many :post, uri: 'posts/for_user/:user_id'
  has_one :image, uri: nil
end
class Post < Spyke::Base
end
user = User.find(3)
# => GET http://api.com/peple/3
user.image
# => Will only use embedded data and never call out to api
user.posts
# => GET http://api.com/posts/for_user/3
Post.find(4)
# => GET http://api.com/posts/4

Post.with('posts/recent')
# => GET http://api.com/posts/recent
Post.with(:recent)
# => GET http://api.com/posts/recent
Post.with(:recent).where(status: 'draft')
# => GET http://api.com/posts/recent?status=draft
Post.with(:recent).post
# => POST http://api.com/posts/recent

Post.find(3).put(:publish)
# => PUT http://api.com/posts/3/publish

Post.request(:post, 'posts/3/log', time: '12:00')
# => POST http://api.com/posts/3/log - { time: '12:00' }

{ title: [{ error: 'blank'}, { error: 'too_short', count: 10 }]}

Article.all 
# => Spyke::ConnectionErorr
Article.with_fallback.all 
# => []
Article.find(1)
# => Spyke::ConnectionError
Article.with_fallback.find(1)
# => nil
article = Article.with_fallback(Article.new(title: "Dummy")).find(1)
article.title
# => "Dummy"

class Article < Spyke::Base
  include_root_in_json true # { article: { title: ...} }
  include_root_in_json :post # { post: { title: ...} }
  include_root_in_json false # { title: ... }
end

class Post < Spyke::Base
  self.connection = Faraday.new(url: 'http://sashimi.com') do |faraday|
    # middleware
  end
end

Started GET "/posts" for 127.0.0.1 at 2018-09-27 02:48:20 +0000
Processing by PostsController#index as HTML
  Parameters: {}
  Spyke (40.3ms) GET http://api.com/posts [200]
Completed 200 OK in 75ms (Views: 64.6ms | Spyke: 40.3ms | ActiveRecord: 0ms)

```


