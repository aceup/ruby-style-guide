# Ruby Style Guide
Ruby Style Guide for Ace-up

These are the style guides we aim to follow when developing new projects in the future and while refactoring existing
code as a part of Tech Debt Reduction.

## Table Of Contents
1. [Comments](#comments)
    1. [File Headers](#file-headers)
    1. [Method Comments](#method-comments)
    1. [Inline Comments](#inline-comments)
    1. [Marker Comments](#marker-comments)
    1. [Commented-Out Code](#commented-out-code)

## Comments
Leave space between the pound sign and the first character unless specified otherwise (ex. Marker Comments)
while writing comments.

### File Headers
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

### Method Comments
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
                                                      content: '<p>Hi ' + coach_name + '</p>
        <p> It was you who wanted to be a coach....deal with it!</p>',
                                                      attachment_url: [''],
                                                      only_for_coach: true
                                                    })
  
      @coach_message.save!
  end
```

### Block and Inline Comments:
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

### Marker Comments
* Marker comments shouldn't have a space between the pound sign and the first character.
* Write comments with proper marker tags to allow easy search:
    * `#TODO` : To mark something which needs to be completed at some point in future.
    * `#FIX` : To mark something which needs to be fixed.
    * `#OPTIMIZE` : To mark something in the code which can be optimized.
    * `#REVIEW` : To mark something which requires review.
    
### Commented-out Code:
* Try to refrain from including commented-out code in the codebase.
    

