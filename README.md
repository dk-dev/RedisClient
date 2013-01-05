# RedisClient

Arduino client for Redis key-value store.

## Features
* Based on Arduino Ethernet Library
* One persistent TCP connection to the Redis server
* Supported different Redis commands (see sourcecode, more to come)
* Very low memory consumption as packets are constructed on Ethernet chip
* Simple parsing of results

## Example Sketch
	IPAddress server(192, 168, 42, 10);
	RedisClient client(server);
	
	void setup() {
		Serial.begin(57600);
		Ethernet.begin(/* add Ethernet options here */);
		client.connect();
		
		// Query a value
		char buffer[100];
		client.GET("samplekey");
		client.resultBulk(buffer, 100);
		Serial.println(buffer);
	}
	
	void loop() {
		Serial.print("Publishing to pub/sub channel... ");
	    client.startPUBLISH("test");
	    client.sendArg("some data");
	    Serial.println(client.resultInt());

	    Serial.print("Getting new ID from Redis... ");
    	uint16_t newid = client.INCR("sequence");
	    Serial.println(newid);
    
	    Serial.print("Adding data to new hash... ");
    	client.startHSET("rawin", newid);
	    client.sendArg("data");
    	client.sendArgRFMData("some data here");
	    Serial.println(client.resultInt());
    
	    Serial.print("Adding timestamp to new hash... ");
    	client.startHSET("rawin", newid);
	    client.sendArg("timestamp");
    	client.sendArg(-1);
	    Serial.println(client.resultInt());

    	Serial.print("Adding new hash to fifo... ");
	    client.startRPUSH("testlist");
    	client.sendArg(newid);
	    uint16_t listitems = client.resultInt();
    	Serial.println(listitems);

	    if (listitems > 100) {
    	  Serial.print("Cutting list to 100 items... ");
	      client.LTRIM("testlist", 0, 99);
    	}
	}