# Ruby Style Guide
##### Ruby Style Guide for Ace-up

* References the [Bozhidar Batsov's Community Driven Ruby Guide](https://github.com/bbatsov/ruby-style-guide) and the
[AirBnb Ruby Style Guide](https://github.com/airbnb/ruby)
* These are the style guides we aim to follow when developing new projects in the future and while refactoring existing
code as a part of Tech Debt Reduction.



## Table Of Contents
1. [General Ace-Up Conventions](#general-rules)
1. [Comments](#comments)
    1. [File Headers](#file-headers)
    1. [Method Comments](#method-comments)
    1. [Inline Comments](#inline-comments)
    1. [Marker Comments](#marker-comments)
    1. [Commented-Out Code](#commented-out-code)
1.  [Code Layout](#code-layout)
1.  [Naming](#naming)
1.  [Classes](#classes)
1.  [Exceptions](#exceptions)
1.  [Collections](#collections)
1.  [Rails](#rails)
    1.  [Scopes](#scopes)
    1.  [Routing](#routing)
    1.  [Controllers](#controllers)
    1.  [Models](#models)
    1.  [ActiveRecord Class Style](#activerecord-class-style)
    1.  [Migrations](#migrations)
    1.  [Mailers](#mailers)
    1.  [Time](#time)
    

### General Ace-Up Conventions
* Always think of corner cases and return an error message to the user to indicate the something went wrong.
* Accessibility-Time Trade-off: Since we work on a consumer facing product so it's important to add checks and fallbacks
to increase user accessibility even if it means that we need to put in extra time to ensure that.


### Comments
* Leave space between the pound sign and the first character unless specified otherwise (ex. Marker Comments)
while writing comments.

#### File Headers
* Every file should have a top-level comment describing what the class does and how it should be used in case of a single
class. 
* For files with multiple classes there should be a descriptive comment for every class and a top-level comment
describing what the file is for. 
  * **Note:** If you're unable to find a common link between the classes in a file then that's a clear indication
  that the classes don't belong in the same file
* For files which have no classes for example initializers or config files, there should still be a top-level comment
describing what the file is for.

    ```ruby
    # Class for auto-generating instructional messages from the Ace-up Bot for Coaches and 
    # Client when a new conversation gets initiated between them.
    class InstructionalBotMessage
      ...
    end
    ```

#### Method Comments
* Write a comment on top of the method to describe what it does.
* Comments should be short and to the point.
* Every method should have comments specifying the input and output unless it is:
    * Short & Obvious 
    * Doesn't interact externally: No initial argument required and no final return
* Don't write unnecessary comments, methods such as `initialize` usually don't need comments.
* **Note:** If a method requires extensive comments, it probably means that the code requires revision.

    ```ruby
    # Call to create instructions for a new conversation
    def self.call(conversation, client)
      self.new(conversation, client).create_instructions
    end
    
    
    def initialize(conversation, client)
      @conversation = conversation
      @client_name = client.user.name
      @coach_name = conversation.coach.user.name
      @ace_up_bot_id = User.find_by_email('support@ace-up.com').id
    end
    
    # Creates a message from Ace-up Bot for the Coach
    def create_coach_instructions
      @coach_message = conversation.messages.build({ user_id: ace_up_bot_id,
                                                      content: content',
                                                      attachment_url: [''],
                                                      only_for_coach: true
                                                    })
    
      @coach_message.save!
    end
    ```

#### Block and Inline Comments:
* Always write comments with the `#` sign even if they are multi-line.
* Describe the different conditions for complex or non-obvious operations, but **don't describe the code itself.**
* Leave a clean line before including an inline comment in a code block.

    ```ruby
    # Check if the pre-created room still exists
    begin
      room = twilio_token.video.rooms(room_name).fetch
    rescue => e
    
      # Twilio returns 20404 for a room which has already been closed.
      if e.code == 20404
      
        # Need to auto-create a new room here with the same name
        room = twilio_token.video.v1.rooms.create(
            unique_name: room_name
        )
      else
        Rails.logger.info("Twilio Video Call room threw an error other than a 404. The error was "+e)
      end
    end
    ```

#### Marker Comments
* Marker comments shouldn't have a space between the pound sign and the first character.
* Write comments with proper marker tags to allow easy search:
    * `#TODO` : To mark something which needs to be completed at some point in future.
    * `#FIX` : To mark something which needs to be fixed.
    * `#OPTIMIZE` : To mark something in the code which can be optimized.
    * `#REVIEW` : To mark something which requires review.
    
#### Commented-out Code:
* Try to refrain from including commented-out code in the codebase.

### Code Layout
* When in doubt about styling for code review the 
[Bozhidar Batsov's Community Driven Ruby Guide](https://github.com/bbatsov/ruby-style-guide)
* Although it should not come in to use a lot because we stick with ASCII characters; but always use **UTF-8** encoding.
* Use two *spaces* or *soft tabs* for indentation.
* Don't use `;` to separate statements or expressions. Follow the Ruby way and write one expression per line.

    ```ruby
    # bad
    puts 'Foo'; puts 'Bar'
    
    # good
    puts 'Foo', 'Bar'
    ```
    
* Use spaces around operators, after commas, colons and semicolons.

    ```ruby
    sum = a + b
    a, b = 1, 2
    class ApplicationController < ActiveController::Base; end
    ```

* **Exceptions to the above rules:**
    * Exponent Operator or rational literals
        ```ruby
        # bad
        val = a + b ** 2
        ratio = 1 / 45
      
        # good
        val = a + b**2
        ratio = 1/45
        ```
    
    * Empty methods can be closed in the same line to allow easier user interpretation.
        ```ruby
        def foo_method; end
        ```
    
### Naming
* `snake_case` for methods and variables.
* `CamelCase` for class names.
* `SCREAMING_SNAKE_CASE` for constants.
* For method and variable names don't separate numbers and characters.

    ```ruby
    # bad
    foo_bar_1 = 1
    
    # good
    foo_bar1 = 1
    ```
* Use `snake_case` for naming files.
* Use `!` only for methods which are dangerous. By dangerous we mean that it can cause damage to components of the
system if used irresponsibly. [Reference](http://davidablack.net/dablog.html#2007/8/15/bang-methods-or-danger-will-rubyist)
* Follow the Ruby naming convention while naming boolean methods.

    ```ruby
    # bad
    def is_cool?
      ...
    end
  
    # good
    def cool?
      ...
    end
    ```

### Classes
* Avoid using class variables `@@` as these can be over-written in inherited Ruby classes.
    ```ruby
    class Foo
      @@some_var = 'foo'
    
      def self.print_var
        puts @@some_var
      end
    end
    
    class Bar < Foo
      @@class_var = 'bar'
    end
    
    Foo.print_var
    # this prints 'bar' instead of `foo`
    ```
* Use *instance* variables instead.
* The class definitions should have a consistent structure. As defined in the ruby style guide:

    ```ruby
    class Person
      # extend and include go first
      extend SomeModule
      include AnotherModule
    
      # inner classes
      CustomError = Class.new(StandardError)
    
      # constants are next
      SOME_CONSTANT = 20
    
      # afterwards we have attribute macros
      attr_reader :name
    
      # followed by other macros (if any)
      validates :name
    
      # public class methods are next in line
      def self.some_method
      end
    
      # initialization goes between class methods and other instance methods
      def initialize
      end
    
      # followed by other public instance methods
      def some_method
      end
    
      # protected and private methods are grouped near the end
      protected
    
      def some_protected_method
      end
    
      private
    
      def some_private_method
      end
    end
    ```
* Define singleton methods using `self`.

### Exceptions
* Don't use exceptions for flow of control. For ex if an Activerecord object doesn't exist then it is an acceptable
else condition and not an exception.

### Collections
* Prefer `%w` to literal array syntax when you need an array of strings.
    ```ruby
    # bad
    BASICS = ['foo', 'bar']
  
    # good
    BASICS = %w[foo bar]
    ```
* Prefer `%i` to literal array syntax when you need an array of symbols.
    ```ruby
    # bad
    BASICS = [:foo, :bar]
    # good
    BASICS = %i[foo bar]
    ```
* Use `Set` instead of `Array` for collection of unique items. `Set` ensures that there are no duplicates.
* Use *symbols* instead of *strings* as hash keys.
    ```ruby
    hash = { 'foo' => 1, 'bar' => 2 }
    hash = { foo: 1, bar: 2 }
    ```
* Prefer `size` over `length` or `count` for performance reasons.
    * `Count` is slower than both ways and used more for enumerable methods. `Size` is better than `length` since
    `length` always loads up new data.

### Strings
* Use interpolation instead of concatenation.

    ```ruby
    # bad
    string = 'Hello' + ' ' + planet.type
  
    # good
    string = "Hello #{planet.type}"
    ```
* Use `\` at the end instead of `+` or `<<` to concatenate multi-line strings.

### Rails
#### Scopes
* Always wrap scopes in lambda
    ```ruby
    # bad
    scope :foo, where(:bar => 1)
    
    # good
    scope :foo, -> { where(:bar => 1) }
    ```
#### Configuration
* Create separate initializer files for every gem with the same name as the name of the gem.
* Keep additional configurations in a `yaml` file in `config`.

#### Routing
* To add more actions to a RESTful resource use `member` or `collection`
    ```ruby
    # bad
    get 'subscriptions/:id/unsubscribe'
    resources :subscriptions
    
    # good
    resources :subscriptions do
      get 'unsubscribe', on: :member
    end
    
    # bad
    get 'photos/search'
    resources :photos
    
    # good
    resources :photos do
      get 'search', on: :collection
    end
    ```
* For multiple `member` or `collection` routes.
    ```ruby
    resources :subscriptions do
      member do
        get 'unsubscribe'
        # more routes
      end
    end
    
    resources :photos do
      collection do
        get 'search'
        # more routes
      end
    end
    ```
* Use nested routes to explain better relationship before ActiveRecord models.
    ```ruby
    class Post < ActiveRecord::Base
      has_many :comments
    end
    
    class Comments < ActiveRecord::Base
      belongs_to :post
    end
    
    # routes.rb
    resources :posts do
      resources :comments
    end
    ```
* Use `namespace` to group related actions together.
    ```ruby
    namespace :admin do
      # Directs /admin/products/* to Admin::ProductsController
      # (app/controllers/admin/products_controller.rb)
      resources :products
    end
    ```
#### Controllers
* Follow the fat models, skinny controllers ideology.
* All business logic should reside in the models, it affects the processing too along with following
the rails conventions.

#### Models
* It's always okay to introduce non-ActiveRecord models. These models perform the functionalities you would expect from
 a Model but don't need to persist data.
* Only put methods that deal with business logic and data-persistance in the models. Other methods belong in helpers.

#### ActiveRecord Class Style
* This references the Rails Style Guide to set an ActiveRecord class style guide.
    ```ruby
    class User < ActiveRecord::Base
      # keep the default scope first (if any)
      default_scope { where(active: true) }
    
      # constants come up next
      COLORS = %w(red green blue)
    
      # afterwards we put attr related macros
      attr_accessor :formatted_date_of_birth
    
      attr_accessible :login, :first_name, :last_name, :email, :password
    
      # Rails4+ enums after attr macros, prefer the hash syntax
      enum gender: { female: 0, male: 1 }
    
      # followed by association macros
      belongs_to :country
    
      has_many :authentications, dependent: :destroy
    
      # and validation macros
      validates :email, presence: true
      validates :username, presence: true
      validates :username, uniqueness: { case_sensitive: false }
      validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
      validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true }
    
      # next we have callbacks
      before_save :cook
      before_save :update_username_lower
    
      # other macros (like devise's) should be placed after the callbacks
    
      ...
    end
    ```
* Prefer `has_many :through` over `has_and_belongs_to_many`. This can be done by creating a `join table` and it allows
additional attributes and validations on the Join table.

    ```ruby
    # not so good - using has_and_belongs_to_many
    class User < ActiveRecord::Base
      has_and_belongs_to_many :groups
    end
    
    class Group < ActiveRecord::Base
      has_and_belongs_to_many :users
    end
    
    # preferred way - using has_many :through
    class User < ActiveRecord::Base
      has_many :memberships
      has_many :groups, through: :memberships
    end
    
    class Membership < ActiveRecord::Base
      belongs_to :user
      belongs_to :group
    end
    
    class Group < ActiveRecord::Base
      has_many :memberships
      has_many :users, through: :memberships
    end
    ```
* If you find using a custom validation more than one time then create a separate file for it and call it in the
required locations. Do the same for `regex` validations.
* Prefer creating named scopes wherever possible.
* The following methods skip model validations, so be cautious while using them:
    * decrement!
    * decrement_counter
    * increment!
    * increment_counter
    * toggle!
    * touch
    * update_all
    * update_attribute
    * update_column
    * update_columns
    * update_counters
* Use `find_each` to iterate over a collection of objects in the database. Looping through a collection of data using
 `all` or `each` is very inefficient since it tries to do batch processing which can lead to high CPU usage.
    ```ruby
    # bad
    Person.all.each do |person|
      person.do_awesome_stuff
    end
    
    Person.where('age > 21').each do |person|
      person.party_all_night!
    end
    
    # good
    Person.find_each do |person|
      person.do_awesome_stuff
    end
    
    Person.where('age > 21').find_each do |person|
      person.party_all_night!
    end
    ```
* Rails creates callbacks for dependent validations so if we specify `before_detroy` after the `dependent: :destroy`
definition then before destroy would only be called once the record has already been destroyed because callbacks are
run in the order of definition. Use `prepend` to prevent this behavior.
    ```ruby
    # bad (roles will be deleted automatically even if super_admin? is true)
    has_many :roles, dependent: :destroy
    
    before_destroy :ensure_deletable
    
    def ensure_deletable
      fail "Cannot delete super admin." if super_admin?
    end
    
    # good
    has_many :roles, dependent: :destroy
    
    before_destroy :ensure_deletable, prepend: true
    
    def ensure_deletable
      fail "Cannot delete super admin." if super_admin?
    end
    ```
* Define the `dependent` option for the `has_many` and `has_one` associations.
* Always use bang! methods when persisting AR objects. This would raise an excaption if applicable. This applies for:
    * `create`
    * `save`
    * `update`
    * `destroy`
    * `first_or_create`
    * `find_or_create_by`
* Favor the use of `find` over `where` when you need to retrieve a single record by id.
* Favor the use of `find_by` over `find_by_attribute` or `where` when you need to retrieve a single record by some
attribute.
* Use `where.not` over SQL.

#### Migrations
* Always commit `Schema.rb` to version control.
* Include default values in the migration itself wherever applicable.
* Enforce foreign-key constraints wherever needed.

#### Mailers
* Name the mailers as `SomethingMailer` so always use the Mailer keyword as suffix.

#### Time
* Don't use `Time.Parse` instead use `Time.zone.parse`
* Don't use `Time.now` instead use `Time.zone.now` or `Time.current`
    


    
