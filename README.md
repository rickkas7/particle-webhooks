# Particle Webhooks Intermediate Tutorial

Particle webhooks are a great way to integrate your Particle Photon or Electron with APIs for web services on the Internet. If you're new to webhooks, you should start at [the official Particle guide for webhooks](https://docs.particle.io/guide/tools-and-features/webhooks/) before continuing on here.

Before we get too far, this tutorial assumes you're familar with [JSON files](http://www.w3schools.com/json/). JSON is a text-based data exchange format for passing data between devices or to and from APIs. As a refresher, the link at the beginning of this paragraph may be helpful.

**Updated March 3, 2017:** There is now information on using the body feature, which allows much more complicated Mustache variable declarations and JSON data structures. 

## Using the Webhook Builder
The basics of the webhook builder are described in the [documentation](https://docs.particle.io/guide/tools-and-features/webhooks/) but I'll include a few handy tips here.

I'll also include the mappings between the fields in the webhook builder and the Custom JSON file here, even though the Custom JSON option is the next section. If you're curious, you can jump ahead and come back. 

![Main Settings Screen](1mainsettings.png)

The *Event Name* field corresponds to what you pass as the `eventName` when you call `Particle.publish` on your Photon or Electron. Note that it is a prefix. So if you have an event name of "test" and the Photon sends "test1" the webhook will be called. You can use this to your advantage for some handy webhook tricks, discussed later. It corresponds to the "event" field in the Custom JSON.

The *URL* field corresponds to the "url" field in the Custom JSON. Query parameters (after the ?) are included in *Query Parameters*, below.

The *Request Type* is the type of request to make the the URL specified above. If you can type the request URL into the browser's URL bar and get a valid response back, it's GET. In most cases, it's POST. It corresponds to the "requestType" field in the Custom JSON and must be one of the strings: "GET", "POST", "PUT", or "DELETE".

The *Device* parameter allows you to choose which device (by its device name) to respond to. This corresponds to the "deviceID" field in the Custom JSON. You can also use the "mydevices" boolean parameter in the Custom JSON; true corresponds to PRIVATE events send from your devices only.

![Custom Data Screen](2customdata.png)

For POST or PUT request types, you can optionally send custom data. There's also default data, described with the *Options* screen, below.

The data can either be in JSON format or URL encoded form format.

In Custom JSON, the "json" field is used for a JSON POST or PUT body. The body will be of type "text/json".

In Custom JSON, the "form" field is used for a URL encoded form. The body will be of type "application/x-www-form-urlencoded".

![Query Parameters Screen](3queryparameters.png)

Query parameters are added to the end of the request URL after a ?. They are a series of key=value pairs that are separated by &.

In Custom JSON, you use the "query" field and set it to an object containing the key/value pairs to set.

![Authentication and Headers Screen](4authheaders.png)

This section is often used for authentication required by the service you're sending to.

If the service requires HTTP Basic Authentication, you can enter the username and password here. In Custom JSON, you set the authorization like this:

```
    "auth": {
    	"username":"testuser",
    	"password":"secret"
    },
```

Often, you can generate an access token for a service, and pass that in a header, such as Authorization. You can do that in the *HTTP Headers* section. This is done by passing a JSON Object of header/value pairs in the "headers" field of the Custom JSON.

![Responses Screen](5responses.png)

The *Response Topic* is event name of the event published with the response. The default topic is "hook-response/*eventName*". You can set this in Custom  JSON using the "responseTopic" field.

The *Error Topic* is the event name of the event published if an error occurs. The default topic is "hook-error/*eventName*". You can set this in the Custom JSON using the "errorTopic" field.

The *Response Template* allows you to format the response data that is published. This is most useful when the server your webhook sends to returns JSON formatted data, as the response topic can contain Mustache templates, described below, and you can pick out only the bits of data that you want to return. You can set this in the Custom JSON using the "responseTemplate" field.

![Options Screen](6options.png)

The *include default data* option is for POST and PUT requests and determines whether to add a common set of default data to JSON or form data. This includes "event" (the event name), "data" (the optional data passed when the event was published), "published_at" (the time the event was sent as an ISO-8601 time string), and "coreid" (the device ID that generated the event. This setting corresponds to the "noDefaults" boolean option in the Custom JSON (Yes=false and No=true).

The *enforce SSL* option determines, for https URLs, whether to validate the server certificate hostname. For public servers this should be true, but if you are sending data to your own home server with a self-signed certificate, you'd use false (and accept a diminished level of security). This setting corresponds to the "rejectUnauthorized" boolean option in the Custom JSON (Yes=true, No=false).


## Using Custom JSON
All of the information included in the webhook builder can be described in a single [JSON file](http://www.w3schools.com/json/). This file can be pasted into the Custom JSON field in the web user interface, or it can be set using the Particle [Command Line Interface](https://docs.particle.io/guide/tools-and-features/cli/photon/) (CLI).

I prefer to use the Particle CLI because I can easily keep multiple versions of my webhooks in files on my computer and upload them with a [single command](https://docs.particle.io/reference/cli/#particle-webhook-create).

One thing to beware of: uploading a hook again doesn't delete an older hook with the same eventName. And having multiple hooks triggering on the same event name can cause massive confusion. Make sure you delete the old hook first if you intended to replace it!

The full list of fields available in the Custom JSON are in the [cloud API documentation](https://docs.particle.io/reference/api/#create-a-webhook).


## Debugging with RequestBin
Depending on the service you're sending data to, it can be difficult to debug a webhook, especially if you're using sending data using templates. A great debugging tool is the free service [http://requestb.in/](http://requestb.in/). You create a new RequestBin and it returns a URL that you use as the URL in your webhook. Then, when you refresh you RequestBin page, it will show you the requests that have come in, with all of the parameters and data. Very handy!

Here's a simple webhook JSON file. I saved it in a file "hook1.json".

```
{
    "event": "test1",
    "url": "http://requestb.in/19le9w61",
    "requestType": "POST",
    "mydevices": true,
    "noDefaults":false
}
```

You can create a webhook using the Particle CLI by issuing the command:

```
particle webhook create hook1.json
```

I tested it first using the CLI:

```
particle publish test1 "testing" --private
```

And this is what the request bin looks like:

![Request Bin Example](bin1.png)

In this simple example, you can see the POST data is in the default format, form data. 

Here's an example using JSON encoding, which we will use more often in the following examples.

```
{
    "event": "test1",
    "url": "http://requestb.in/19le9w61",
    "requestType": "POST",
    "json": {
		"name": "{{PARTICLE_EVENT_NAME}}",
		"value": "{{PARTICLE_EVENT_VALUE}}"
    },
    "mydevices": true,
    "noDefaults": true
}

```

Generating the event:

```
particle publish test1 "testing2" --private
```

And the RequestBin results:
![Request Bin Example](bin2.png)

## Mustache

One of the great features of the webhook is using mustache/handlebars templates. It allows you to do simple processing of JSON data both in the request data going out and response data coming back. It's a logic-less templating system so you're not actually writing code, but you can do useful and powerful text transformations.

We were actually using mustache in the example above:

```
"json": {
	"name": "{{PARTICLE_EVENT_NAME}}",
	"value": "{{PARTICLE_EVENT_VALUE}}"
},
```

The `{{PARTICLE_EVENT_VALUE}}` is a mustache template to include the event value in the data. It's automatically replaced by the event value when the template is processed.

The other values you can use are: `{{PARTICLE_DEVICE_ID}}` and `{{PARTICLE_PUBLISHED_AT}}`.

But that's a really simple example. We can do powerful parsing of JSON data, which will be discussed below in the topics on sending and receiving of JSON data. 

Let's say you have this JSON data:

```
{
	"a": {
		"aa":"testing",
		"ab":1234
	},
	"b": "xxx",
	"c": [
		{
			"ca":"first",
			"cb":123
		},
		{
			"ca":"second",
			"cb":456
		}
	]
}
```

Here are some example Mustache templates and what they substitute:

```
template: {{a.aa}}
result:   testing

template: {{a.ab}}
result:   1234

template: {{b}}
result:   xxx

template: {{c.0.ca}}
result:   first

template: {{c.1.cb}}
result:   456
```

You can test these out yourself using the Mustache Tester described below. It can greatly simplify making mustache definitions!

In other words, you separate nested elements by "." and access array elements by just using their array index (0-based).

If you're encoding strings that may contain HTML special characters like angle brackets, ampersand, and some other characters, you may need to use a triple curly bracket like `{{{b}}}` instead. This is described below.


If you're already familiar with mustache, there are some additional restrictions that don't apply to generic mustache templates used in, say, [HTML](https://www.scaler.com/topics/html/). The hook JSON template must itself be valid JSON. You can get around these limitations by using the body, described below.

The template below is not valid in the json section. Even if the event value is a number once the substition is complete, and would be valid then, you still cannot do this because the template itself is not valid JSON *before* substitution. **Don't do this:**

```
"json": {
	"value": {{PARTICLE_EVENT_VALUE}}
},
```

It must always be in a string:

```
"json": {
	"value": "{{PARTICLE_EVENT_VALUE}}"
},
```

However, there is a workaround that can be used to get around this limitation by using the body definition, described in the next section.

You can use Mustache templates in the url, query, json, form, and body elements.

Incidentally, it's called mustache because of the use of curly brackets, which sort of look like a mustache turned sideways.

## Using the body definition

In addition to `json` and `form` to specify the body of a POST (or PUT) method, you can also use `body`. The body allows free-form definition of the data. This is useful if your server requires a different, non-JSON and non-form format, but we'll be using it here to output complex JSON data.

You currently can only use a body definition from a webhook in a file using the Particle CLI. You can't enter one into the console.

In these examples we'll be using the body definition to send complex JSON data to a server. You may want to add a content type definition so your knows you're sending JSON data.

```
    "headers": {
    	"Content-Type": "text/json"
    },
```

Here's an example webhook:

```
{
    "event": "test1",
    "url": "http://requestb.in/195kr671",
    "requestType": "POST",
    "mydevices": true,
    "noDefaults":false,
    "body":"{\"a\":{{a}}}"
}
```

Note that the body parameter is JSON-escaped string string. This means that you need to enclose it in double quotes, and escape any double quotes in your string. This results in a slightly confusing mess:

```
    "body":"{\"a\":{{a}}}"
```

It's better to think of these definitions in the unescaped version. I also added a spaces to make it clearer:

```
{ "a":{{a}} }
```

This solves the numbers in the JSON data problem above. If a is a number, you get an actual number. This works for integers and floating point numbers. If your Photon was sending the following data in a publish:

```
{"a":1234.5, "b":"abcdef"}
```

The webhook request body would contain:

```
{"a":1234.5}
```

While you can use this technique for strings, it's only practical for simple strings. For example:

```
{ "a":"{{a}}" }
```

This works fine if you pass in a simple string:

```
particle publish test1 '{"a":"xxx"}' --private
```

But if your string contains backslashes or double quotes, things start to go downhill quickly, since the body processor doesn't know it's generating JSON, it doesn't know how to escape the special JSON characters and will likely just give up and pass an empty string to the webhook.

Also, in many cases, you want want to use triple-curly brackets instead. The reason is that Mustache by default HTML-escapes strings, which is almost never what you want when creating JSON.

```
particle publish test1 '{"a":"<x>"}' --private
```

with the template above produces:

```
{"a":"&lt;x&gt;"}
```

If you use triple curly brackets

```
{ "a":"{{{a}}}" }
```

You get the expected:

```
{"a":"<x>"}
```

This affects anything that will be HTML escaped, like angle brackets, the ampersand, and sometimes some characters like double quotes that get converted to HTML entities like `&quot;`.

### Expanding key names

Sometimes you'll be interfacing with an API that has verbose keys. Google likes to do this. The problem is that publishes are limited to 255 characters, and sometimes you can run out of room if you compose the JSON on the Photon/Electron. Instead, you can use mustache templates to expand key names.

In this fragment of body template (without the escaping of double quotes)

```
{"cellId":{{i}},"locationAreaCode":{{l}},"mobileCountryCode":{{c}},"mobileNetworkCode":{{n}} }
```

You'd send up in your publish data like:

```
{"i":1234,"l":567,"c":890,"n":765}
```

and the template would convert it to:

```
{"cellId":1234,"locationAreaCode":567,"mobileCountryCode":890,"mobileNetworkCode":765}
```

Remember that double quotes need to escaped in your webhook JSON, so it will look more like this in your webhook JSON file:

```
"body":"{\"cellId\":{{i}},\"locationAreaCode\":{{l}},\"mobileCountryCode\":{{c}},\"mobileNetworkCode\":{{n}} }"
```

### Passing pre-formatted arrays

Sometimes you might want to pre-encode part of your JSON webhook data. A good reason is that you have a variable-length array of data.
Say you're passing the following up via publish:

```
{"a":[123,456,789]}
```

And you use this Mustache template in your webhook body (before string escaping):

```
{"anArray":[{{a}}]}
```

Then the output would be:

```
{
    "anArray": [
        123,
        456,
        789
    ]
}
```

The other reason you might want to do this is to mix in some pre-defined things, like:

```
{"anArray":[{{a}}], "id":"{{PARTICLE_DEVICE_ID}}"}
```

This would output:

```
{
    "anArray": [
        123,
        456,
        789
    ],
    "id": "12345678901234567890abcd"
}
```

You can also use this technique to implement a "zero or more" option. Using the same template above, this input:

```
{"a":123}
```

Generates a valid JSON array in the output:

```
{
    "anArray": [
        123
    ],
    "id": "12345678901234567890abcd"
}
```

And if there is no a property at all, then you get an empty array.


### Body with conditional blocks

Conditional blocks can be used for optional objects. This would allow you to use a single webhook for multiple functions.

If you have this published value:

```
{"a":123,"b":{"c":"hello","d":false}}
```

And this mustache template:

```
{ "cat":"{{b.c}}", "dog":{{b.d}}, "apple":{{a}} }
```

You get this JSON output:

```
{
    "cat": "hello",
    "dog": false,
    "apple": 123
}
```

Using a conditional block, you can also implement it using this mustache template:

```
{
	{{#b}}
		"cat":"{{c}}", 
		"dog":{{d}}, 
	{{/b}}
	"apple":{{a}} 
}
```

This generates the same output as above. The `{{#b}}` tests if b exists and if it does, includes the statements until the `{{/b}}`. Also, any mustache variables within the block are relative to b. That's why cat is `{{c}}` in this example and `{{b.c}}` in the example before that.

The real difference is when you only publish this:

```
{"a":123}
```

The output won't have any references to the cat and dog at all:

```
{
    "apple": 123
}
```

### Body with arrays

This technique can also be used to handle arrays of varying size, with certain caveats and some annoying complexity.

The data you pass up by publish contains a JSON object, which contains a JSON array, which contains more objects. The length of the array may vary.

```
{"a":[{"b":123,"c":true},{"b":456,"c":false}] }
```

And you have this template:

```
{
	"array":[
		{{#a}}
		{
			"banana":{{b}},
			"capybara":{{c}}
		},
		{{/a}}
		{}
	]
}
```

Like the conditional block example above, this template uses `{{#a}}`. However, since it's an array, the text until `{{/a}}` is repeated once for every element in the array. This allows for easy expansion of objects contained in an array.

And this is what gets generated:

```
{
    "array": [
        {
            "banana": 123,
            "capybara": true
        },
        {
            "banana": 456,
            "capybara": false
        },
        {            
        }
    ]
}
```

Unfortunately, there's a problem. Because mustache was designed to generate HTML, not JSON, it doesn't really know what to do about the comma separator needed between JSON array elements.

There are three ways to deal with this:

* Leave the extra comma after the last element. 

If the template had been this:

```
{
	"array":[
		{{#a}}
		{
			"banana":{{b}},
			"capybara":{{c}}
		},
		{{/a}}
	]
}
```

You'd get this:

```
{
    "array": [
        {
            "banana": 123,
            "capybara": true
        },
        {
            "banana": 456,
            "capybara": false
        },
    ]
}
```

Note the comma before the ]. This isn't valid JSON, but the thing you're sending your webhook data to may be OK with it, and if so then you're set.

* Add an empty element

That's what's done above. Each array element ends with a comma, but then there's an element with no data at the end. Some servers may be OK with this, which is valid JSON, but weird data.

```
		},
		{{/a}}
		{}
```

* Duplicate the first element again

```
{
	"array":[
		{{#a}}
		{
			"banana":{{b}},
			"capybara":{{c}}
		},
		{{/a}}
		{
			"banana":{{a.0.b}},
			"capybara":{{a.0.c}}
		}
	]
}
```

This outputs:

```
{
    "array": [
        {
            "banana": 123,
            "capybara": true
        },
        {
            "banana": 456,
            "capybara": false
        },
        {
            "banana": 123,
            "capybara": true
        }
    ]
}
```

I used this technique with the Google geolocation API, which doesn't like the first two methods, but does not mind the duplicated element.

## Using the Mustache Tester

The syntax of mustache is obscure enough that you'll probably want to use the mustache tester instead of creating webhook after webhook trying to get it right. 

The tester is written in Javascript, runs entirely in your web browser, and is here:

[http://rickkas7.github.io/mustache/](http://rickkas7.github.io/mustache/)

- Enter JSON data to parse

Paste your JSON data into this box. Typically this is a JSON object, like this:

```
{"a":1234.5, "b":"abcdef"}
```

It can also be an array, or a complex object, with a mix of objects and arrays.

- Show formatted JSON data

If you select the checkbox a formatted version of the JSON data is displayed. This is handy if your JSON is all in one line and contains nested objects and arrays. It expands these out so it's much easier to see the nesting.

If you click on a row of formatted JSON, it displays the Mustache template you can use to extract that data. The **Copy To Template** button will append that selector to your template.

- Enter a mustache template to test

Paste your mustache template here. 

- Processed template and data

If you have valid JSON data and a valid mustache template, this is the result. If you were creating a webhook, this is what would be in the JSON body. It doesn't have to be JSON, however. If you are using a response template you might want to use something simpler like comma-separated values, for example.

- Processed template and data, as formatted JSON

If the processed template and data are valid JSON, the formatted version is put here. This is a good sanity test to make sure you are generating valid JSON, assuming that's what you're aiming for.

- String-escaped version of the template

When using the body definition you may find this useful. It does the escaping of double quotes so you can paste it right into your webhook JSON file. 

The checkbox below enables or disables removing CR, LF and tab from the string-escaped template. Normally they're not necessary in the body template, and this makes it easier to paste readable text in the box and get a more compact representation


## Sending Complex Data
In the examples above we pass a simple string in the "value" field in JSON. But what if you have multiple pieces of data you want to send to the server? Mustache can help you there too!

In the Google Elevation API example below we sent data from the Photon in JSON format. 

The Photon code looks like this:

```
float lat = 39.73915360;
float lng = -104.98470340;

char data[256];
snprintf(data, sizeof(data), "{\"lat\":%f, \"lng\":%f}", lat, lng);

Particle.publish("getElevation", data, PRIVATE);
```

This will sent the data up to the cloud formatted like this:

```
{"lat":39.73915360, "lng":-104.98470340}
```

Then, in your query and json webhook templates, you can now use `{{lat}}` and `{{lng}}` Mustache variables to access the values of those fields.

Also handy: If you need to generate an access token on your Particle device, you can pass it up along with other data in JSON format, then include it in an Authorization header or in a URL query parameter using Mustache.


## Receiving Complex Data
While webhooks can return larger response data in multiple pieces, this is a pain to deal with on the Photon or Electron. Fortunately, you can use mustache templates to parse a JSON response returned by the server and only return the data you're interested in. You can also use it to turn response data into easier to parse formats, like comma-separated values.


## Example: Google Elevation API

This is an example of using mustache templates in both the request and response. It uses the [Google Maps Elevation API](https://developers.google.com/maps/documentation/elevation/start) to do a simple query.

According to the Google documentation, you make a request like this:

```
https://maps.googleapis.com/maps/api/elevation/json?locations=39.7391536,-104.9847034&key=YOUR_API_KEY
```

And you get a response back in JSON, like this:

```
{
   "results" : [
      {
         "elevation" : 1608.637939453125,
         "location" : {
            "lat" : 39.73915360,
            "lng" : -104.98470340
         },
         "resolution" : 4.771975994110107
      }
   ],
   "status" : "OK"
}
```

Hook definition (hook.json):

```
{
    "event": "getElevation",
    "url": "https://maps.googleapis.com/maps/api/elevation/json",
    "requestType": "GET",
    "query": {
		"locations": "{{lat}},{{lng}}",
		"key": "<paste your secret Google API key here>"
    },
    "responseTemplate": "{{results.0.elevation}}",
    "mydevices": true,
    "noDefaults": true
}
```

Since this a GET request we use the "query" in the hook template.

The Google API wants a parameter "location" with a value of two decimal numbers for the latitude and longitude, separated by a comma. We can do this easily using the Mustache template `{{lat}},{{lng}}`.

In the response we want to return just the "elevation" parameter. The returned JSON object contains a key "results" that is an array. In the array is an object with the "elevation". Rather than dealing with parsing the whole result on the Photon we return just the elevation using the template `{{results.0.elevation}}`.

Particle Code:

```
#include "Particle.h"

bool requestSent = false;

void hookResponseHandler(const char *event, const char *data);


void setup() {
	Serial.begin(9600);

	Particle.subscribe("hook-response/getElevation", hookResponseHandler, MY_DEVICES);
}

void loop() {
	if (!requestSent && millis() > 10000) {
		requestSent = true;

		float lat = 39.73915360;
		float lng = -104.98470340;

		char data[256];
		snprintf(data, sizeof(data), "{\"lat\":%f, \"lng\":%f}", lat, lng);

		Serial.printlnf("sending request %s", data);

		Particle.publish("getElevation", data, PRIVATE);
	}
}

void hookResponseHandler(const char *event, const char *data) {
	float elevation = atof(data);

	Serial.printlnf("elevation=%f", elevation);
}
```

The code is pretty simple. The thing to note is that we format the data parameter to the Particle.publish call in JSON format so it can be intepreted by the webhook. 

In this simple example, we could just also just put the comma-separated values in data and used `{{PARTICLE_EVENT_VALUE}}` but doing it in JSON is a better general-purpose solution that can be used in more situations.


Output in the serial monitor:

```
sending request {"lat":39.739155, "lng":-104.984703}
elevation=1608.609985
```

And what it looks like in the event log in the dashboard:

![Event Log](eventlog.png)


## Example: Google Firebase

I created a separate tutorial for using Google Firebase and webhooks for storing data from Photons, Electrons, etc..

There are three examples in the tutorial:

- Saving data in a table
- Saving data in a per-device table
- Saving data from a device and retrieving it from another device

You don’t need your own server; it’s all done with webhooks and the regular Google Firebase API. And if you don’t have much data to exchange, you may be able to use the free tier forever.

Also, it demonstrates using mustache templates and separating data by device name.

[https://github.com/rickkas7/firebase_tutorial](https://github.com/rickkas7/firebase_tutorial)




