---
layout: page
---

<h1 class="no-margin-top">Multichannel chat server</h1>

This example mount a multichannel chat server
using Sandstone topic creation feature.


## Installation

Create your composer.json:

{% include file-title.html filename="composer.json" %}
{% include highlight-file.html filename="examples/multichannel-chat/composer.json" code="json" %}

Then run `composer update`.


## Server script

You will first need to create the topic class.

This class must extends `Eole\Sandstone\Websocket\Topic`,
and then implements these methods:

- `onSubscribe`: called when someone join this topic
- `onPublish`: called when someone publish a message to this topic
- `onUnSubscribe`: called when someone unsubscribes

The logic behing broadcasting messages to all subscribing client of a topic is a logic
that differs depending on what you want to implement.

In our case, the logic would be:

- `onSubscribe`: *I want to receive message from this topic*
- `onPublish`: *I send a message on this topic*
- `onUnSubscribe`: *I don't want to receive messages from this topic anymore*

In other hand, the `Eole\Sandstone\Websocket\Topic` class provides a method
to broadcast a message to every subscribing client, example:

``` php
$this->broadcast([
    'type' => 'message',
    'message' => $event,
]);
```

Then we have enough to create the `ChatTopic` class:

{% include file-title.html filename="ChatTopic.php" %}
{% include highlight-file.html filename="examples/multichannel-chat/ChatTopic.php" code="php" %}

This class will be used for a chat topic, i.e `general` or `technical`.

To create one topic, this could do the job:

``` php
$app->topic('chat/general', function ($topicPattern) {
    return new ChatTopic($topicPattern);
});
```

What will happens here:

When a client subscribes to `chat/general` topic,
Sandstone will call your callback to get a new instance of your `ChatTopic`,
and provides the string `"chat/general"` to your instance.

This same instance will be used for each future client subscribing to this topic.

In the same way, a message `Someone has joined this channel.` will be broadcasted
to all subscribing clients on new subscriptions.

When someone send a message, it is just broadcasted to every subscribing clients.

But we don't want to only one channel, we want to allow to create dynamic channels:

``` php
$app->topic('chat/{channel}', function ($topicPattern) {
    return new ChatTopic($topicPattern);
});
```

This works the same way as Silex routes.

So let's create the server script:

{% include file-title.html filename="chat-server.php" %}
{% include highlight-file.html filename="examples/multichannel-chat/chat-server.php" code="php" %}

Now the chat server works.


## How to test it with javascript

There is a Javascript library,
[Autobahn|JS](http://autobahn.ws/js/reference_wampv1.html),
which provides an implementation of the WAMP protocol.

> **Note**:
> Be careful to use the **0.8.x** version of the library
> in order to work with WAMP **v1**.

Following the documentation, we can use our chat with:

{% include file-title.html filename="front-end.html" %}
``` html
{% include examples/multichannel-chat/front-end.html %}
```

Then, run your chat server:

``` bash
php chat-server.php
```

And go to this page. You should receive chat messages in your Javascript console.
