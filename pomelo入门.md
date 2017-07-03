# pomelo入门

#### Protocol.encode 
- 加密  
-   
	Protocol.encode = function(id,route,msg){  
		var msgStr = JSON.stringify(msg);  
		if (route.length>255) { throw new Error('route maxlength is overflow'); }  
		//强类型数组  
		var byteArray = new Uint16Array(HEADER + route.length + msgStr.length);  
		var index = 0;  
		byteArray[index++] = (id>>24) & 0xFF;  
		byteArray[index++] = (id>>16) & 0xFF;  
		byteArray[index++] = (id>>8) & 0xFF;  
		byteArray[index++] = id & 0xFF;  
		byteArray[index++] = route.length & 0xFF;  
		//先把他们穿华为unicode码  
		for(var i = 0;i<route.length;i++){  
		byteArray[index++] = route.charCodeAt(i);  
		}  
		for (var i = 0; i < msgStr.length; i++) {  
		byteArray[index++] = msgStr.charCodeAt(i);  
		}  
		//将这些unicode码转化为字符串  
		return bt2Str(byteArray,0,byteArray.length);  
	};
- id 客户端提供
- rotete 发送目的地
- 发送内容  json格式 {}
- 前5个int是头部，头部的前4个是id，第5个表示数据表示route的长度，然后剩下的保存要发送的数据  
- 头部-route-内容

#### pomelo.init
	- 
	pomelo.init = function(params, cb){
		    pomelo.params = params;
		    params.debug = true;
		    var host = params.host;
		    var host = "121.199.40.246";   //主机地址
		    var port = params.port;   //主机端口
		
		    var url = 'ws://' + host;
		    if(port) {
		      url +=  ':' + port;
		    }
		//进行websocket的链接
		    socket = io.connect(url, {'force new connection': true, reconnect: false});
		
		    socket.on('connect', function(){
		      console.log('[pomeloclient.init] websocket connected!');
		      if (cb) {
		        cb(socket);
		      }
		    });
		
		    socket.on('reconnect', function() {
		      console.log('reconnect');
		    });
		    //如果收到了数据，那么调用processMessage来处理这些数据
		    socket.on('message', function(data){
		      if(typeof data === 'string') {
		        data = JSON.parse(data);
		      }
		      if(data instanceof Array) {
		        processMessageBatch(pomelo, data);
		      } else {
		        processMessage(pomelo, data);
		      }
		    });
		
		    socket.on('error', function(err) {
		      console.log(err);
		    });
		
		    socket.on('disconnect', function(reason) {
		      pomelo.emit('disconnect', reason);
		    });
	 };
- 建立websocket的连接，然后设置一些监听函数

#### pomelo.request
	- 
	//用于向pomelo后台发送数据，route是要发送到的地方
	//接下来的参数可以是要发送的数据，以及消息确认的回调函数
	  pomelo.request = function(route) {
	    if(!route) {
	      return;
	    }
	    var msg = {};
	    var cb;
	    arguments = Array.prototype.slice.apply(arguments);
	    if(arguments.length === 2){
	      if(typeof arguments[1] === 'function'){
	        cb = arguments[1];
	      }else if(typeof arguments[1] === 'object'){
	        msg = arguments[1];
	      }
	    }else if(arguments.length === 3){
	      msg = arguments[1];
	      cb = arguments[2];
	    }
	    msg = filter(msg,route);
	    id++;   //当前发送的数据的id
	    callbacks[id] = cb;   //当发送的这个数据有返回的时候，用这个回调函数来处理
	    var sg = Protocol.encode(id,route,msg);   //将数据转化为规定的格式
	    socket.send(sg);
	  };
- 参数 route，msg，cb
