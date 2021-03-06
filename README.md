# socketcluster-client-ruby
A Ruby client for socketcluster.io

In `lib/socketclusterclient`, you'll find the files you need to be able to package up your Ruby library into a gem. Put your Ruby code in the file . To experiment with that code, run `bin/console` for an interactive prompt.

Refer below examples for more details.

Overview
--------
This client provides following functionality

- Easy to setup and use
- Support for emitting and listening to remote events
- Automatic reconnection
- Pub/sub
- Authentication (JWT)
- Can be used for extensive unit-testing of all server side functions
- Support for ruby >= 2.2.0

Installation
------------

Add this line to your application's Gemfile:

```ruby
gem 'socketclusterclient'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install socketclusterclient

Usage
-----------
Create instance of `Socket` class by passing url of socketcluster-server end-point

```ruby
    # Create a socket instance
    socket = ScClient.new('ws://localhost:8000/socketcluster/')
```
**Important Note** : Default url to socketcluster end-point is always *ws://somedomainname.com/socketcluster/*.

#### Registering basic listeners

- Different functions are given as an argument to register listeners

```ruby
    require 'socketclusterclient'
    require 'logger'

    logger = Logger.new(STDOUT)
    logger.level = Logger::WARN

    on_connect = -> { logger.info('on connect got called') }

    on_disconnect = -> { logger.info('on disconnect got called') }

    on_connect_error = -> { logger.info('on connect error got called') }

    on_set_authentication = lambda do |socket, token|
      logger.info("Token received #{token}")
      socket.set_auth_token(token)
    end

    on_authentication = lambda do |socket, is_authenticated|
      logger.info("Authenticated is #{is_authenticated}")
    end

    socket = ScClient.new('ws://localhost:8000/socketcluster/')
    socket.set_basic_listener(on_connect, on_disconnect, on_connect_error)
    socket.set_authentication_listener(on_set_authentication, on_authentication)
    socket.connect
```

#### Connecting to server

- For connecting to server

```ruby
    # This will send websocket handshake request to socketcluster-server
    socket.connect
```

- By default reconnection to server is disabled, so to enable automatic reconnection

```ruby
    # This will set automatic-reconnection to socketcluster-server repeat it infinitely
    socket.set_reconnection(true)
```

- To set delay of reconnecting to server

```ruby
    # This will set automatic-reconnection to socketcluster-server with delay of 2 seconds and repeat it infinitely
    socket.set_delay(2)
    socket.connect
```

- To disable automatic reconnection to server

```ruby
    # This will disable reconnection to socketcluster-server
    socket.set_reconnection(false)
```

- For reconnection to the server you need to set reconnection strategy

```ruby
    # This will set the reconnection strategy
    socket.set_reconnection_listener(reconnect_interval, max_reconnect_interval, max_attempts)

    # default reconnection strategy
    socket.set_reconnection_listener(2000, 30_000, nil)
```

For example,
```ruby
    socket.set_reconnection_listener(3000, 30_000, 10)  # (reconnect_inverval, max_reconnect_interval, max_attempts)
    socket.set_reconnection_listener(3000, 30_000)  # (reconnect_inverval, max_reconnect_interval)
```

- You can set reconnect_interval, max_reconnect_interval and max_attempts directly as well
```ruby
    socket.reconnect_interval = 2000
    socket.max_reconnect_interval = 20_000
    socket.max_attempts = 10
```

- To allow reconnection to server, you need to reconnect

```ruby
    # This will set automatic-reconnection to socketcluster-server with delay of 2 seconds and repeat it till the maximum attempts are finished
    socket.connect
```


Emitting and listening to events
--------------------------------

#### Event emitter

- To emit a specified event on the corresponding socketcluster-server. The object sent can be String, Boolean, Long or JSONObject.

```ruby
    # Emit an event
    socket.emit(event_name, message);

    socket.emit("chat", "Hi")
```

- To emit a specified event with acknowledgement on the corresponding socketcluster-server. The object sent can be String, Boolean, Long or JSONObject.

```ruby
    # Emit an event with acknowledgment
    socket.emitack(event_name, message, ack_emit)

    socket.emitack('chat', 'Hello', ack_emit)

    ack_emit = lambda do |key, error, object|
      puts "Got ack data => #{object} and error => #{error} and key => #{key}"
    end
```

#### Event Listener

- To listen an event from the corresponding socketcluster-server. The object received can be String, Boolean, Long or JSONObject.

```ruby
    # Receive an event
    socket.on(event, message)

    socket.on('ping', message)

    message = lambda do |key, object|
      puts "Got data => #{object} from key => #{key}"
    end
```

- To listen an event from the corresponding socketcluster-server and send the acknowledgment back. The object received can be String, Boolean, Long or JSONObject.

```ruby
    # Receive an event and send the acknowledgement back
    socket.onack(event, message_with_acknowledgment)

    socket.onack('ping', ack_message)

    ack_message = lambda do |key, object, block_ack_message|
      puts "Got ack data => #{object} from key => #{key}"
      block_ack_message.call('error lorem', 'data ipsum')
    end
```

Implementing Pub-Sub via channels
---------------------------------

#### Creating a channel

- For creating and subscribing to channels

```ruby
    # Subscribe to channel
    socket.subscribe('yell')

    # Subscribe to channel with acknowledgement
    socket.subscribeack('yell', ack_subscribe)

    ack_subscribe = lambda do |channel, error, _object|
      puts "Subscribed successfully to channel => #{channel}" if error == ''
    end
```

- For getting list of created channels

```ruby
    # Get all subscribed channel
    channels = socket.subscribed_channels
```

#### Publishing an event on channel

- For publishing an event

```ruby
    # Publish to a channel
    socket.publish('yell', 'Hi')

    # Publish to a channel with acknowledgment
    socket.publishack('yell', 'Hi', ack_publish)

    ack_publish = lambda do |channel, error, _object|
      puts "Publish sent successfully to channel => #{channel}" if error == ''
    end
```

#### Listening to channel

- For listening to a channel event

```ruby
    # Listen to a channel
    socket.onchannel('yell', channel_message)

    channel_message = lambda do |key, object|
      puts "Got data => #{object} from key => #{key}"
    end
```

#### Un-subscribing to channel

- For unsubscribing to a channel

```ruby
    # Unsubscribe to a channel
    socket.unsubscribe('yell')

    # Unsubscribe to a channel with acknowledgement
    socket.unsubscribeack('yell', ack_unsubscribe)

    ack_unsubscribe = lambda do |channel, error, _object|
      puts "Unsubscribed to channel => #{channel}" if error == ''
    end
```

Logger
------

#### Disabling logger

- To get logger for logging of events

```ruby
    socket.logger.info("Some information")
    socket.logger.warn("Some warning")
```

- To disable logging of events

```ruby
    socket.disable_logging
```

- To enable logging of events

```ruby
    socket.enable_logging
```

Development
-----------

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

Contributing
------------

Bug reports and pull requests are welcome on GitHub at https://github.com/opensocket/socketcluster-client-ruby. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

License
-------

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).
