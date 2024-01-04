---
layout: post
title: No More Refreshing!
date: 2023-05-12    
categories: Rails ActionCable WebSockets
---

_This post was first published on the [Brewer Digital Engineering Blog](https://brewerdigital.com/engineering/no-more-refreshing-intro-to-websockets-and-actioncable-in-rails/)_

<h1>No More Refreshing! – Intro to WebSockets and ActionCable in Rails</h1>

<h2>WebSockets and ActionCable Overview</h2>

Have you ever been on a website that you had to keep reloading just to get new information? It can be frustrating, right? Well, WebSockets can fix that! They’re a technology that allows websites to communicate with a server in real-time, so updates can be pushed to your screen as soon as they happen – without you having to constantly hit refresh. In this post, I’ll dive into what WebSockets are, how they work, and how you can use them to make your web apps more dynamic and interactive.

<h3>WebSockets</h3>

WebSocket is a type of web protocol that allows for an open connection allowing for continuous communication being a client and a server, as opposed to the one-and-done style of HTTP. You have probably experienced WebSockets when doing any sort of live activity on the internet, such as using a chat room or watching live sports updates.

<h3>ActionCable</h3>

ActionCable is the Ruby on Rails implementation of WebSockets and allows a quick and easy set-up for adding WebSockets to your Rails applications. Using a generator you can get up and running with the core classes needed to start running WebSocket connections. To start, use the built-in command line Rails Generator:

```
rails generate channel TeamUpdatesChannel
```

And Rails will generate a spec file, the new channel file, and a connection file (if there isn’t one yet). Let’s take a look at what these all are.

<h3>What are Channels?</h3>

If you are familiar with Ruby on Rails applications, channels will feel very similar to your controllers. They hold all the actions that you want that channel (which is most similar to an API endpoint) to handle. These classes inherit from `ApplicationCable::Channel`, identically to how a controller inherits from `ApplicationController`. So you can include any shared functionality in the parent and all the child controllers will have access to those methods.

<h3>Channel Methods</h3>

Just how a controller will have standard actions, like `show`, `index`, or `create`, Channels have a few standard methods:

- `subscribed`: used to create the subscription to the stream
- `unsubscribed`: used to handle any clean up you may want to manage when the client unsubscribes from the stream
- `receive`: used to handle incoming data from a client

Channels have various ActionCable methods to hook up all the WebSocket subscriptions for you, handling the subscription and the broadcast. You will create new subscriptions in the channel class, however, you can trigger broadcasts from anywhere within your Rails application.

<h4>Stream_from</h4>

`stream_from(broadcasting_name)`: used to subscribe the client to the name of the broadcast passed in the argument. The name of the broadcast can be whatever you want it to be, like a string literal `“premier_league_live_scores”` or something more dynamic like `“#{params[:team]}_live_scores”`

Then to broadcast to the subscribed clients you call the broadcast method with the broadcasting name and the data you want to send from anywhere within your application.

```
ActionCable.server.broadcast(“premier_league_live_scores”, { body: “Kick-off at The Emirates“ })
```

<h4>Stream_for</h4>

`stream_for(object)`: Very similar to `stream_from` (it actually calls `stream_from` [under the hood](https://api.rubyonrails.org/v7.0.4.2/classes/ActionCable/Channel/Streams.html#method-i-stream_for)). Used to subscribe the client to broadcast for a model:

```
class TeamUpdatesChannel < ApplicationCable::Channel
  def subscribed
    team = Team.find(params[:id])
    stream_for team
  end
end
```

Then to broadcast to all the subscribed clients you would call the broadcast method on the Channel class itself passing the model and the data to be sent like so:

```
TeamUpdatesChannel.broadcast(@team, @score)
```

<h3>What is Connection.rb?</h3>

This connection class mainly handles authentication and authorization. If you have used `ApplicationController` to handle authentication and set a variable like `current_user` before, then this will feel very similar.

```
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_user(request.params[:token])
      if self.current_user
        return self.current_user
      else
        reject_unauthorized_connection
      end
    end

    private

    def find_user(auth_token)
      User.find_by(auth_token:)
    end
  end
end
```

<h3>Connection Methods</h3>

As seen above, the connection class has some standard helpful methods built in:

`connection#connect`: the primary instance method that you will define in the class. This method runs when establishing a WebSocket connection and is therefore where you can store authentication and authorization logic.

`reject_unauthorized_connection`: used to reject the WebSocket connection in case your authorization fails. This will close the connection and return a 404

`identified_by`: helps create an identifier for the connection. As the example demonstrates this is a great way to share user information, as your channels will have access via a method of the same name as the identifier (e.g. `current_user`)


<h3>RSpec Testing</h3>

RSpec comes with ActionCable testing right out of the box. It has several methods and matchers available to use in your specs that will help test all the features I have demonstrated so far.

<h4>Methods</h4>

`stub_connection`: stubs the connection object and sets `identified_by` methods

```
user = User.first
stub_connection(current_user: user)
```

`subscribe`: calls the subscribed method for the channel you are testing

```
team = Team.first
subscribe(id: team.id)
```

`perform(:method, args)`: used to call a channel method (like `receive`). Call it in an `expect` block to the check the outcomes

```
expect { perform(:receive, message) }
    .to have_broadcasted_to(team)
    .from_channel(PremierLeagueLiveScoresChannel)
```

<h4>Matchers</h4>

```
RSpec.describe TeamUpdatesChannel, type: :channel do
  it 'subscribes to the stream' do
    team = Team.first
    subscribe(id: team.id)

    expect(subscription.to be_confirmed)
    expect(subscription).to have_stream_for(proposal)
  end
end
```

Testing is a core of any reliable Rails application, RSpec has several fairly self-explanatory matchers that can test the standard functionality your channels will likely be performing:

`expect(subscription).to be_confirmed`: used to check the subscription was successfully created

`expect(subscription).to be_rejected`: used to check the subscription was rejected

`expect(subscription).to have_stream_from(channel_name)`: used to check the subscription is streaming from the correct channel

`have_broadcasted_to(stream_name)`: used to check that the channel broadcasted to the correct stream

`from_channel(channel_name).with(message_data)`: can be chained to the above `have_broadcasted_to` matcher. Used to check the from channel and the data being broadcasted

<h2>Wrap-up</h2>
I have found WebSockets to be a huge boon to applications that benefit from, or require, a live user experience. ActionCable brings the power of WebSockets to Ruby on Rails applications with the same ease of use that Rails developers already know and love. For more info check out the official ActionCable [docs](https://guides.rubyonrails.org/action_cable_overview.html)!
