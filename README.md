# Wolox on Rails - Authentication
[![Build Status](https://travis-ci.org/Wolox/wor-authentication.svg)](https://travis-ci.org/Wolox/wor-authentication)
[![Code Climate](https://codeclimate.com/github/Wolox/wor-authentication/badges/gpa.svg)](https://codeclimate.com/github/Wolox/wor-authentication)
[![Test Coverage](https://codeclimate.com/github/Wolox/wor-authentication/badges/coverage.svg)](https://codeclimate.com/github/Wolox/wor-authentication/coverage)
[![Issue Count](https://codeclimate.com/github/Wolox/wor-authentication/badges/issue_count.svg)](https://codeclimate.com/github/Wolox/wor-authentication)

Easily authenticate entities in your application with tokens that can be renewed before they expire and which must be re-created after a maximum customizable amount of days. Also, add your own validations when creating, renewing and invalidating tokens!

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'wor-authentication'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install wor-authentication

## Usage

### Basic configuration

As we have already said, this gem lets you authenticate entities in your application. To get that done, we must define some controller from which all other controllers will have to extend to have only-authenticated routes. So, let's do that in our `ApplicationController.rb`:
```ruby
class ApplicationController < ActionController::Base
  include Wor::Authentication::Controller
  before_action :authenticate_request
end
```
> To know which exceptions can be thrown by the gem, please check SOME_EXCEPTION_FILE.

Now that we have done this, we have to define the routes to achieve authentication and a controller to handle them.
```ruby
# routes.rb
Rails.application.routes.draw do
  # your routes go here
  post '/' => 'authentication#create'
  post '/renew' => 'authentication#renew'
  post '/invalidate_all' => 'authentication#invalidate_all'
end

# authentication_controller.rb
class AuthenticationController < ApplicationController
  include Wor::Authentication::SessionsController
  skip_before_action :authenticate_request, only: [:create]
end
```
> Note that our controller extends from ApplicationController.

Finally, we have to add a `secret_key_base` key to our  `secrets.yml` file.

### User tracking and custom validations

#### Validations before giving out a token? Override `authenticate_entity`:

```ruby
# authentication_controller.rb
def authenticate_entity(params)
  user ||= User.find_by(email: params[:email])
  return nil unless user.present? && user.valid_password?(params[:password])
end
```
> Returning no value or false won't create the authentication token.

#### Keeping track of users? Override: `entity_payload`:

```ruby
# authentication_controller.rb
ENTITY_KEY = :user_id

def entity_payload(user)
  { ENTITY_KEY => user.id }
end

def find_authenticable_entity(entity_payload_returned_object)
  User.find_by(id: entity_payload_returned_object.fetch(ENTITY_KEY))
end
```

#### Validations in every request? Override `entity_custom_validation_value` to get it verified as the following:

```ruby
# authentication_controller.rb
def entity_custom_validation_value(user)
   user.some_value_that_shouldnt_change
end
```
> If it is desired to update this value when renewing the token, override: `entity_custom_validation_renew_value`.

#### Invalidating all tokens for a user? Override `entity_custom_validation_invalidate_all_value` as the following:

```ruby
# authentication_controller.rb
def entity_custom_validation_invalidate_all_value(user)
   user.some_value_that_shouldnt_change = 'some-new-value'
   user.save
end
```
> This works only if `entity_custom_validation_value` has been overridden.


### Some other useful configurations

#### Want to modify tokens ttl? Override `new_token_expiration_date` as the following:

```ruby
def new_token_expiration_date
  (Time.zone.now + 2.days).to_i
end
```

#### Want to modify tokens maximum useful day? Override `token_maximum_useful_date` as the following:

```ruby
def token_maximum_useful_date
  (Time.zone.now + 30.days).to_i
end
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Run rubocop lint (`rubocop -R --format simple`)
5. Run rspec tests (`bundle exec rspec`)
6. Push your branch (`git push origin my-new-feature`)
7. Create a new Pull Request

## About ##

This project is maintained by [Alejandro Bezdjian](https://github.com/alebian) and it was written by [Wolox](http://www.wolox.com.ar).
![Wolox](https://raw.githubusercontent.com/Wolox/press-kit/master/logos/logo_banner.png)

## License

**wor-authentication** is available under the MIT [license](https://raw.githubusercontent.com/Wolox/wor-authentication/master/LICENSE.md).

    Copyright (c) 2016 Alejandro Bezdjian, aka alebian

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in
    all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.
