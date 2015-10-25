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

	 app.post('/upload',[ multer({ dest: './uploads/'}), function(req, res){
	
	   if( req.files.image )
		 {
 		 fs.readFile( req.files.image.path, function (err, data) {
 	  		if (err) throw err;
   			var img = new Buffer(data).toString('base64');
			client.lpush("images", img)
 			});
 		}
	
		res.send("Uploaded!")
	 }]);

	app.get('/meow', function(req, res)
 	{
		client.lpop("images", function(err, value){
			if(value){
				res.writeHead(200, {'content-type':'text/html'});
				res.write("<img src='data:my_pic.jpg;base64,"+value+"'/>");
	    		res.end();
			}
			else{
				res.send("No more images!")
			}
		});

 	});
 	
### Additional service instance running

	 var server = app.listen(3000, function () {
	
	   var host = server.address().address
	   var port = server.address().port
	   console.log('Server1 listening at http://%s:%s', host, port)
		 client.lpush("servers", "http://"+host+":"+port)
	 })
	
	 var server1 = app.listen(3001, function () {
	
		 var host = server1.address().address
		 var port = server1.address().port
		 console.log('Server2 listening at http://%s:%s', host, port)
		 client.lpush("servers", "http://"+host+":"+port)
	 })

### Demonstrate proxy

	var proxyServer = proxy.listen(3002, function (){
	
		 var host = proxyServer.address().address
		 var port = proxyServer.address().port
		  console.log('Proxy server listening at http://%s:%s', host, port)
	 })
	 
	 proxy.use(function(req, res, next){
		client.rpoplpush("servers", "servers", function(err, reply){
			if(err) throw err;
			console.log("Redirecting to "+reply)
			res.redirect(reply+req.url);
		})
	});

