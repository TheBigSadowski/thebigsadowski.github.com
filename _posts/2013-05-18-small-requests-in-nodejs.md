---
layout: post
title: Faster Small Requests in Node.js
---


# Faster Small Requests in Node.js

The other day I ran into a bit of an issue with a small node.js web server that would return a small response (in my case a transparent pixel for reporting activity in real-time) when it was accessed from a relatively high-latency location, say the other side of the country or across the Pacific Ocean. If you write your code in a nieve way, you can end up doubling the number of trips exchanges that need to take place between the client and server, resulting in response times where transfer time looks the same as waiting time.

![response timing write() and then end()](/img/2013-05-18-small-requests-in-nodejs/write-then-end.png)

## The slow code

We will start with a simple application that only returns the text "Hello World!"

The server is pretty simple, it just writes "Hello World!" and then ends the request. It should work fine, but as we will see, over high-latency links this will likely take twice as long for the data to get to the user as opposed to just sending the response with the end() method.

    var http = require('http');
    http.createServer(function (req, res) { 
        res.writeHead(200, {'content-type': 'text/html; charset=UTF-8'});
        res.write('<h1>Hello World!</h1>');
        res.end();
    }).listen(8888);

To test all this, I set up a vm (Ubuntu 13.04) in Singapore and ran the code above. When hitting this service from San Francisco, I get about 200ms of transfer time to get the first byte and headers and then another 200ms or so to get the actual "Hello World!" content and then end of the response. For this test, I'm just running it in the browser (Chrome) and established the connection previously so we would just be measuring the time to make the request. Here are the results:

![response timing write() and then end()](/img/2013-05-18-small-requests-in-nodejs/write-then-end.png)

Here's what's happening:
1. The call to `writeHeader()` is cached by node until the first body data is sent (or `end()` is called). It does this to send the first chunk and the headers in the same network packet.
2. The call to `res.write("<h1>Hello World!</h1>")` sends the response.
3. The response is received, but not properly ended with an empty chunk, so the client confirms it received the first packet and asks for the next one
4. The server responds with the next packet that is basically just an indicator that there is no more data to receive.

What that means is that we're going round-trip to the server twice to get data that we could have easily received in the first trip.

Now, if we replace the separate calls to `res.write("<h1>Hello World!</h1>")` and `res.end()` with one call to `res.end("<h1>Hello World!</h1>")` with the full response, node.js will send the whole response back at once.

Here's the code:

    var http = require('http');
    http.createServer(function (req, res) { 
        res.writeHead(200, {'content-type': 'text/html; charset=UTF-8'});
        res.end('<h1>Hello World!</h1>');
    }).listen(8888);

When we run this version on a far away server, the response looks a lot better:

![response timing write() and then end()](/img/2013-05-18-small-requests-in-nodejs/just-end.png)

We've managed to push the whole response back at once and now the client doesn't have to make another round trip to the server!

### Another way to speed it up

We could also speed up the first code by telling node in the call to `writeHeader` what the content length is. This informs node.js that we know how long the request is and that the response should not be sent in chunks (the default behavior). 

    var http = require('http');
    http.createServer(function (req, res) { 
        res.writeHead(200, {'content-type': 'text/html; charset=UTF-8', 'content-length': 21});
        res.write('<h1>Hello World!</h1>');
        res.end();
    }).listen(8888);

This also has the benefit of reducing the amount of data sent in the response by a few bytes because the chunk envelope is not there saving us 6 bytes in this case, not having the closing chunk saves us 5 bytes and "content-length: 21" is a shorter header than "transfer-encoding: chunked" saving us another 8 bytes, for a total savings of 19 bytes over the chunked response.


### Why does it work this way?

The node.js developers made a choice early on to not get in between your code and the internal workings of the internet like a lot of other we frameworks do. One of the choices they made was to not buffer output and immediately send it down to the client. Therefore, that first call to `write()` actually fires off a packet to the client and then the call to `end()` is buffered because the socket is busy. Only when the client returns saying the first packet was delivered will it finally end the response.

So, the moral of the story...  when sending tiny responses in node.js and speed is a top priority, build up your response and then pass your whole response in the call to `end()`.



### What if we don't use chunked encoding:


    var http = require('http');
    http.createServer(function (req, res) { 
        res.writeHead(200, {'content-type': 'text/html; charset=UTF-8', 'content-length': 21});
        res.write('<h1>Hello World!</h1>');
        res.end();
    }).listen(8888);

