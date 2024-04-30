**Project: Microservices architecture and system design** 


(I have shared all my learnings from this project in file named lessons_learned.txt in this repository only.)

This documentation outlines the workflow of our system, which handles user authentication, file uploads, conversion, and notifications. The process is divided into multiple services, each with its unique role.

![micro1](https://github.com/bhanumalhotra123/systemdesign-k8s/assets/144083659/c6492572-7bac-4464-a849-ddac44452d0e)


![micro2](https://github.com/bhanumalhotra123/systemdesign-k8s/assets/144083659/4ccccfbc-820b-4025-8524-1aa782b140bf)


![micro3](https://github.com/bhanumalhotra123/systemdesign-k8s/assets/144083659/3f80fdbb-8886-44ae-94c3-17cf40ff28de)





Steps:

1.Gateway Service:

When a user enters the system, the Gateway Service checks if they have provided a valid username and password.
If valid, the Gateway Service sends a request to the Authentication Service to verify the credentials against our MySQL database.
Upon successful verification, a JWT token is generated and returned to the user.
```

$ curl -X POST http://mp3converter.com/login -u bhanucorrect@gmail.com:devops                                                                                   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   193  100   193    0     0   9146      0 --:--:-- --:--:-- --:--:--  9650eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImJoYW51Y29ycmVjdEBnbWFpbC5jb20iLCJleHAiOjE2OTM5MjExNjMsImlhdCI6MTY5MzgzNDc2MywiYWRtaW4iOnRydWV9.woEtLjr1d6Nvh9pIu7Li3PAvY72XYcpOwv03XxBkaIs
```

2.File Upload and Authentication:

When a user wants to upload a file, they present their JWT token to the Gateway Service.
The Gateway Service, once again, contacts the Authentication Service to ensure the integrity of the token and prevent tampering.

After successful validation, a connection to RabbitMQ is established, and the message is placed in a queue for processing.

Simultaneously, the file is saved to MongoDB for storage.


3.Converter Service:

The Converter Service continuously consumes messages from the RabbitMQ queue, processing them one by one.
For each message, it queries MongoDB for the corresponding video file using the provided ID.
The Converter Service converts the video file and stores the result in another MongoDB database, named 'mp3.'
It then places a message on the queue, indicating the ID of the converted file.
```
$ curl -X POST -F 'file=@./test.mkv' -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImJoYW51Y29ycmVjdEBnbWFpbC5jb20iLCJleHAiOjE2OTM5MjExNjMsImlhdCI6MTY5MzgzNDc2MywiYWRtaW4iOnRydWV9.woEtLjr1d6Nvh9pIu7Li3PAvY72XYcpOwv03XxBkaIs' http://mp3converter.com/upload
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 1218k  100     8  100 1217k     40  6142k --:--:-- --:--:-- --:--:-- 6182ksuccess!

```

Notification Service:

The Notification Service monitors the queue for messages containing converted file IDs.
When a message is detected, it sends a notification to the user, informing them that the MP3 file with the specified ID is ready for download.


4.File Download:

To download the converted file, the user provides the ID of the desired MP3 file through the Gateway.
The Authentication Service once again verifies the user's credentials.
Upon successful authentication, the Gateway retrieves the file from MongoDB and provides it to the user for download.

This workflow ensures secure user authentication, seamless file upload and conversion, timely notifications, and secure file retrieval. It's designed to provide a smooth user experience while maintaining data integrity and security at each step of the process.

```
HP@bhanumalhotra MINGW64 ~/Desktop/microservices-k8s/src/converter (main)
$ curl --output abc.mp3 -X GET -H 'Authorization: Bearer  eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6ImJoYW51Y29ycmVjdEBnbWFpbC5jb20iLCJleHAiOjE2OTQwMDY2ODcsImlhdCI6MTY5MzkyMDI4NywiYWRtaW4iOnRydWV9.Ir_VUxYBPYIxhnOTHZE0nu4w1phOR5D7EuhlNYUaYSU' "http://mp3converter.com/download?fid=64f72cf3a67ec20c1ee18817"  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    14  100    14    0     0    213      0 --:--:-- --:--:-- --:--:--   218

HP@bhanumalhotra MINGW64 ~/Desktop/microservices-k8s/src/converter (main)
$ ls
Dockerfile  abc.mp3  consumer.py  convert/  manifests/  requirements.txt  test.mkv  venv/
```
