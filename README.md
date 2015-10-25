Cache, Proxies, Queues
=========================

### Set/Get

		app.get('/get', function(req, res) {
		  client.get("string key", function (err, reply) {
			if(reply)
			{
				client.ttl('string key', function writeTTL(err, data) {
					res.send(reply.toString()+" expires in "+ data);
				})
			}
			else
				res.send("Key Expired!")
			});
		})
		
		app.get('/set', function(req, res) {
		
		  client.set("string key", "This message will self-destruct in 10 seconds", redis.print);
			client.expire("string key", 10);
			client.ttl('string key', function writeTTL(err, data) {
				res.send('Key inserted with TTL '+ data);
			})
		})

### Recent sites

	app.use(function(req, res, next)
	{
		//console.log(req.method, req.url);
	
		client.lpush("rec", req.url, function(err,value){
			if(err)
				console.log(err);
		});
		 client.ltrim("rec", 0, 4);
	
		next(); // Passing the request to the next handler in the stack.
	});
	
	
	app.get('/recent', function(req, res) {
		 client.lrange("rec",0,4,function(err,value){
			 res.send(value)
		 });
	});
	
	
### Upload/Meow

You will be using [redis](http://redis.io/) to build some simple infrastructure components, using the [node-redis client](https://github.com/mranney/node_redis).

	var redis = require('redis')
	var client = redis.createClient(6379, '127.0.0.1', {})

In general, you can run all the redis commands in the following manner: client.CMD(args). For example:

	client.set("key", "value");
	client.get("key", function(err,value){ console.log(value)});

### An expiring cache

Create two routes, `/get` and `/set`.

When `/set` is visited, set a new key, with the value:
> "this message will self-destruct in 10 seconds".

Use the expire command to make sure this key will expire in 10 seconds.

When `/get` is visited, fetch that key, and send value back to the client: `res.send(value)` 


### Recent visited sites

Create a new route, `/recent`, which will display the most recently visited sites.

There is already a global hook setup, which will allow you to see each site that is requested:

	app.use(function(req, res, next) 
	{
	...

Use the lpush, ltrim, and lrange redis commands to store the most recent 5 sites visited, and return that to the client.

### Cat picture uploads: queue

Implement two routes, `/upload`, and `/meow`.
 
A stub for upload and meow has already been provided.

Use curl to help you upload easily.

	curl -F "image=@./img/morning.jpg" localhost:3000/upload

Have `upload` store the images in a queue.  Have `meow` display the most recent image to the client and *remove* the image from the queue.

### Proxy server

Bonus: How might you use redis and express to introduce a proxy server?

See [rpoplpush](http://redis.io/commands/rpoplpush)
