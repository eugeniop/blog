---
layout: post
title: "OTA Updates for Arduino Projects - Part III"
date:   2021-09-25 21:22 -0800
categories: arduino ota sdu updates 
comments: true
author: Eugenio Pace
---

A second step in getting OTA updates to the Arduino board is getting the bits downloaded. I added downloading capabilities to the `HTTPRequest` implementation.

First step is to detect it is a file coming down the wire. I do that by checking for the `Content-Disposition` header presence:

```c++
...
if(strncmp(line, "Content-Disposition", 19)==0){
	strtok(line, "\"");
	const char * fileName = strtok(NULL, "\"");
	strcpy(response.fileName, fileName);
	response.file = 1;
	return; 
}
...
```

Some quick and dirty parsing gives me the `fileName`. For this implementation, I always assume `8.3` filename format. I realize the `strcpy` is kind of dangerous, but since I control both ends, I am ok.

`response.file` is a simple flag that then tells the parser a file is going to be downloaded. (`response` is a simple class to represent the HTTP Response).

```c++
HTTPResponse * processFileDownload(){
	
	//Initialize SD
	if(!SD.begin(SD_CS)){
		return NULL;
	}

	File file = SD.open(response.fileName, O_RDWR | O_CREAT);
	file.seek(0); // Write from the beginning

	int bytesWritten = 0;
	int bytesReady = 0;
	while(bytesReady = client.available()){
		/*
			This is an arbitrary buffer. It looks like the WiFi card can return anything between
			500-1000 bytes
			Serial.print(bytesReady);
		*/
		char data[700];
		int r = client.readBytes(data, 700);
		file.write(data, r);
		bytesWritten += r;
	}

	file.close();
	client.stop();

	//Did we get everything?
	if(bytesWritten == response.length){
		return &response;
	}

	//Truncated?
	return NULL;
}
```

A few noteworthy things:

1. `client.available()` returns the number of bytes ready to read. I am only using this variable to get debug information.
2. I'm using a buffer to get a chunk of bytes. I've experimented with 200-1000 bytes. Somewhere ~700 was delivering good throughput for me. Your mileage might vary.
3. I've avoided dynamically allocating a buffer because this is simpler. I also have a buffer ready to use in the helper `HTTPResponse` object.
4. I check that the actual total bytes I read matches the size reported in the `HTTPResponse.length` field. In all the experiments I've ran, I've never seen data being truncated, but ... better be safe.
5. I'm considering adding some kind of integrity validation, perhaps a checksum of sorts? I'm not sure this is needed yet.  
6. I experimented with a few files of various sizes, from a few KB to 200KB (about the end size of the sketch). It worked quite well in all cases, and I was greatly surprised at the speed.
7. Given how well this works, perhaps I can use the file transfer approach to do other things? (e.g. download offline content seems like a good candidate)

In the next part, I will focus on building the binary outside of the IDE. 
