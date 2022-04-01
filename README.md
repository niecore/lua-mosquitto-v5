lua-mosquitto-v5
================

> :warning: **This is a fork**: Link to stale [Pull request](https://github.com/flukso/lua-mosquitto/pull/33)

Lua bindings to the [libmosquitto](http://www.mosquitto.org/) client library.

The parameters to all functions are as per [libmosquitto's api](http://mosquitto.org/api)
only with sensible defaults for optional values, and return values directly
rather than via pointers.

Generated API documentation for the lua functions [is also available](docs/)

Compile
-------
You need Lua and mosquitto development packages (headers and libs) to
build lua-mosquitto.

Compile with

    make

You can override the pkg-config package name to set a specific Lua version.
For example:

    make LUAPKG=lua5.2

Example usage
-------------

Here is another mosquitto v5 example

```Lua
mqtt = require("mosquitto")
client = mqtt.new()
client:option(mqtt.OPT_PROTOCOL_VERSION, mqtt.MQTT_PROTOCOL_V5);

client.ON_CONNECT_V5 = function(success, rc, rc_string, flags, properties)
        client:subscribe_v5("v5")
end

client.ON_SUBSCRIBE_V5 = function(mid, properties, ...)
	-- send own
	properties = {}
	properties["content-type"] = "text/json"
	properties["response-topic"] = "this/is/my/response/topic"
	properties["payload-format-indicator"] = 0
	properties["user-property"] = {a = "testA", b = "testB"}
	properties["message-expiry-interval"] = 255
	assert(mqtt:publish_v5("v5", "message", 0, false, properties))
end

client.ON_MESSAGE_V5 = function(mid, topic, payload, qos, retain, properties)
	print("ON_MESSAGE_V5" .. properties["response-topic"]) 
end

broker = arg[1] -- defaults to "localhost" if arg not set
client:connect(broker)
client:loop_forever()
```


Here is a very simple example that subscribes to the broker $SYS topic tree
and prints out the resulting messages:

```Lua
mqtt = require("mosquitto")
client = mqtt.new()

client.ON_CONNECT = function()
        print("connected")
        client:subscribe("$SYS/#")
        local mid = client:subscribe("complicated/topic", 2)
end

client.ON_MESSAGE = function(mid, topic, payload)
        print(topic, payload)
end

broker = arg[1] -- defaults to "localhost" if arg not set
client:connect(broker)
client:loop_forever()
```

Here is another simple example that will just publish a single message,
"hello", to the topic "world" and then disconnect.

```Lua
mqtt = require("mosquitto")
client = mqtt.new()

client.ON_CONNECT = function()
        client:publish("world", "hello")
        local qos = 1
        local retain = true
        local mid = client:publish("my/topic/", "my payload", qos, retain)
end

client.ON_PUBLISH = function()
	client:disconnect()
end

broker = arg[1] -- defaults to "localhost" if arg not set
client:connect(broker)
client:loop_forever()
```

