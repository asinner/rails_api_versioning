# API Versioning in Rails
This document contains some different approaches for API versioning in Rails.

### Put the version in the URL
- Namespace your routes:
```ruby
namespace :api do
  namespace :v1 do
    resources :articles
  end
  
  namespace :v2 do
    resources :articles
  end
end
```

- Then create your controllers:
```ruby
module Api::V1
  class ArticlesController
    #
  end
end
```
- Your new endpoints:
  - /api/v1/articles
  - /api/v2/articles

#### Pros/Cons
Pros:
  - Simpler than creating a new mime-type and specifying version number in header
Cons:
  - Some consider putting the version numbers in the URL

### Send the version in the Accept header
The following solution comes from an [article](http://www.bignerdranch.com/blog/adding-versions-rails-api/) by Jay Hayes. This approach is basically as follows:

- Set up the constraints in the routes file:
```ruby
scope module: :v1, constraints: ApiConstraint.new(version: 1) do
  resources :articles, only: :index
end
```
This creates V1 routes for the API.

- Create your controller for the API version you are building:
```ruby
module V1
  class ArticlesController < ApplicationController
    def index
      articles = [
        { id: 123, name: 'The Things' },
      ]
 
      respond_to do |format|
        format.json do
          render json: articles
        end
      end
    end
  end
end
```
Rinse and repeat the previous two steps for future API versions.

- Create the contsraint in app/constraints:
```ruby
# app/constraints/api_constraint.rb

class ApiConstraint
  attr_reader :version
 
  def initialize(options)
    @version = options.fetch(:version)
  end
 
  def matches?(request)
    request
      .headers
      .fetch(:accept)
      .include?("version=#{version}")
  end
end
```
The constraint requires that the version number is sent in the 'Accept' header in the request. 
- Define a new mime type:

```ruby
# config/initializers/mime_types.rb
Mime::Type.register 'application/vnd.articles+json', :articles_json
```
- Respond with that format in your controllers:
```ruby
respond_to do |format|
  format.articles_json # Your response here
end
```
#### Pros/cons
Pros:
  - Considered a best practice
Cons:
  - A little bit heaver than the URL version approach
