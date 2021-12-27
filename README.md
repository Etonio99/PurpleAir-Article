# Making API Calls with the PurpleAir API

This guide will help you understand the basics of sending HTTP requests and making API calls. This article assumes a basic knowledge of programming.

## Introduction
All sensor data reported from PurpleAir sensors are able to be collected from our database using the PurpleAir API. The API follows REST principles and uses the methods GET, POST, PUT, and DELETE. PurpleAir’s API documentation can be viewed at api.purpleair.com.

The PurpleAir API uses two different API keys; a read key and a write key, which need to be used to make specific API calls. If you want to use the PurpleAir API yourself, you can request your own keys by contacting us. Your first and last name are necessary for us to assign keys to you.

The data collected with the API will include the Raw PM values and not the AQI. Since there are so many different ways to display air quality data, any conversion of the raw data to AQI values will need to be performed.

## Make Your First API Call
There are three parts to making an API request; the *request line*, *header*, and *body*.

**Request Line**: The *request line* will first include one of the four REST methods mentioned previously; GET, POST, PUT, and DELETE. A GET request is used to collect data entries. A POST request is used to create data entries. A PUT request is used to update existing data entries. A DELETE request is used to remove existing data entries.

A URL will follow the request method. The URL is the web address we want to send a request to. For the PurpleAir API, the URL will start with https://api.purpleair.com/v1/. What comes after that will depend on the type of request we want to make.

**Header**: Next comes the *header*. The *header* holds the context for our request. When using the PurpleAir API, this will always include your respective API key for the specific call you want to make. The parameter for this is named “X-API-Key”.

**Body**: Last of all is the *body*. The *body* will include the payload that we want to send through the API. The only requests that will require a *body* are POST and PUT requests.

With all three of these parts put together we have a complete request. The last thing to do is to send it.

Let’s look at an example of a request to check what type of API Key we are sending. This example will be completed using Curl.

curl -X GET https://api.purpleair.com/v1/keys -H 'X-API-Key: ********-****-****-****-************'
Here you can see some of the parts of a request that were mentioned earlier. This request uses GET as the request method and https://api.purpleair.com/v1/keys as the URL, and together they make up the *request line*, which is preceded by -X when using Curl.

The *header* of this request is 'X-API-Key: ********-****-****-****-************', which is recognized in cURL by a string that follows a -H. One of your API keys would replace the line of asterisks. Either key would work here, but for this example we will use a read key.

Since there is no data that we will need to send to the URL, a *body* does not need to be included.

Then, inside of a terminal, we can put in our request and it will return something very similar to this:

{
  "api_version" : "V1.0.10-0.0.12",
  "time_stamp" : 1609459200,
  "api_key_type" : "READ"
}
This is the data that our request has collected. It returned three separate items; the current version of the API, the time this request was completed, and the type of API key we sent. As you can see, since we put in a read key, the “api_key_type” it returned was “READ”. If we had entered a write key, the request would have returned “WRITE”.

## Collect Sensor Data
Now that we’ve been able to successfully complete an API call, let’s try one slightly more complex. This next request will be used to collect the real-time data of any specific public sensor on the PurpleAir map.

curl -X GET https://api.purpleair.com/v1/sensors/***** -H 'X-API-Key: ********-****-****-****-************'
This request is very similar to the last one that checked the API key, but notice that the URL in this request ends with /sensors instead of /keys. This part of the URL will change depending on what data we want to collect. After /sensors, you will also see a short line of asterisks. Here you will put the sensor ID of the sensor you want to collect data from. This can be found by clicking on a Map Marker on the PurpleAir map and then checking the short 4-6 character code in the URL that follows “select=”.

We will need to use a read API key here, since we are going to be collecting a data entry.

If we now put this request inside of a terminal and send it, it will return something similar to this:

{
  "api_version" : "V1.0.10-0.0.12",
  "time_stamp" : 1609459200,
  "data_time_stamp" : 1609460400,
  "sensor" : {
      "..."
  }
}
You may notice the beginning of the return looks very similar to the return we got from our last request, but after that follows a “data_time_stamp” variable and a “sensor” variable that will include a lot of information that I have replaced here with an ellipsis ‘…’. More information on the data that is returned in this call can be seen in the PurpleAir API documentation under Sensors > Get Sensor Data > Sensor data fields.

## Use of Groups
### Create a New Group
If you are planning on querying data from multiple sensors at once, we ask that instead of sending an individual request for each sensor, you send a single request that queries all of them. You can request data from multiple sensors at once by using groups.

A group is an array of sensors under a specific name and ID that is linked to a set of API keys. When making a call to create a group you must assign it a name. Once it is created, it will be given its own ID. Then, a user can populate that group with sensors, or members, as they are referred to in the documentation. Once a group is populated, it can be queried in a single request and that request will return that data of all of the members included within it. Groups are assigned to the API keys they are created with.

Here is an example Curl request to create a group named “My Group”:

curl -X POST https://api.purpleair.com/v1/groups -H 'Content-Type: application/json' -H 'X-API-Key: ********-****-****-****-************' -d '{"name":"My Group"}'
As we have seen before this request contains a *request line* and a *header*. However it also includes a parameter that starts with -d. This is the *body* and the ‘d’ stands for data. What follows is the payload that we are going to be sending in our request. As mentioned previously to create a group you must assign it a name. This is done by including the following JSON in the payload; '{"name":"My Group"}'. You can change the name of the group to anything you feel appropriate, but for this example we will leave it as “My Group”.

Once we put in and enter this command, it will return something similar to the following example:

{
  "api_version" : "V1.0.10-0.0.12",
  "time_stamp" : 1609459200,
  "group_id": 001
}
As usual, you can see the API version and the time stamp are included, but afterwards you are able to see a new variable. This is the group ID that is now assigned to the group you have just created. It is the ID you will use to query data from this new group.

If you ever need to see the IDs of the groups you have created, you can query those by using the Get Groups List request, and if you want to see the individual members within a specific group, you can do that using the Get Group Detail request.

### Populate a Group With Members
Now that we have created a group we can populate it with members. The request we are going to use to do this is the Create Member request.

Here is another example Curl request that adds a member:

curl -X POST https://api.purpleair.com/v1/groups/***/members -H 'Content-Type: application/json' -H 'X-API-Key: ********-****-****-****-************' -d '{"sensor_index":"*****"}'
In the URL of this request we, will need to include the group ID of the group we want to add our sensor to and then follow that with /members. Since we are going to be adding a data entry we are going to input our API write key, and lastly we will input the corresponding sensor index for the sensor we want to add.

Once that is completed we can put it into a terminal and run it.

{
  "api_version" : "V1.0.10-0.0.12",
  "time_stamp" : 1609459200,
  "data_time_stamp" : 1609460400,
  "group_id" : 001,
  "member_id" : 001,
  "sensor" : {
    "..."
  }
}
This return will appear very similar to the output from the getSensorData request but also includes two new values. These values are the group-id and the member_id. The group_id is the ID if the group the sensor was added to and the member-id is a new ID that was created specifically for the sensor to be accessed within the set group.

### Query Data From a Group
Now that we have a group with members, we can collect sensor data from that group as a whole. We can do this using the Get Member Data request and the Get Members Data request.

The following example will use the Get Member Data request:

curl -X GET https://api.purpleair.com/v1/groups/***/members/*** -H 'X-API-Key: ********-****-****-****-************'
As you can see the URL in this request will require both the group ID and the member ID of the sensor that you want to query. Then in the *header* we will include our API read key. Running this command will give an identical return to the Create Member request we demonstrated previously. It will return the appropriate time stamps, group and member IDs, and the sensor data.

With the Get Members Data request you would be able to query any amount of specified fields from all of the members within a specific group. Here we see an example of an API call collecting the longitude and latitude values of the members within a group:

{
  "api_version" : "V1.0.10-0.0.12",
  "time_stamp" : 1609459200,
  "data_time_stamp" : 1609460400,
  "group_id" : 001,
  "max_age" : 604800,
  "firmware_default_version" : "6.01",
  "fields" : [
    "sensor_index",
    "longitude",
    "latitude"
  ],
  "data" : [
[00001,41.183660,-112.598448],
[00002,47.762003,-87.824733]
  ]
}
Here we see all of the fields we input will be returned to us in a variable named data. The sensor index of each sensor queried will always appear first in each entry, followed by the specified fields we entered.

## Collect Data From a Private Sensor
The requests that we have sent so far will work only for public PurpleAir sensors. If you would like to collect data from a private sensor, we will have to access it a bit differently. Instead of being able to use the getSensorData request, we will have to first add the private sensor to a group. Then within the group you can query it for the data you want to collect.

To add a private sensor to a group, we are going to use the Create Member request again, but with different parameters.

Here is an example Curl request that does that:

curl -X POST https://api.purpleair.com/v1/groups/***/members -H 'Content-Type: application/json' -H 'X-API-Key: ********-****-****-****-************' -d '{"sensor_id":"**:**:**:**:**:**", "owner_email": "example@email.com"}'
Similar to the earlier group requests, this request has all three parts; a *request line*, a *header*, and a *body*. However you may note that this request has two *headers* included. We have added another *header* here to tell the server what type of data we are going to be sending, which in this context is JSON, hence the addition of -H 'Content-Type: application/json'.

As well, in the *body* of the request you can see that we have included two separate values; the sensor_id and the owner_email. The sensor-id is different from what we had input previously, which was the sensor_index. The ID we want to us now is the Device-ID (MAC address) of the sensor we want data from. The Device-ID is a unique code for every sensor that is printed on the sensor sticker and listed in the shipping confirmation email.

For the owner_email value we will input the email that was used in the Owner’s Email field on the registration form.

Now that that information has been input, we can put in our request and send it.

{
  "api_version" : "V1.0.10-0.0.12",
  "time_stamp" : 1609459200,
  "data_time_stamp" : 1609460400,
  "group_id" : 001,
  "member_id" : 001,
  "sensor" : {
    "..."
  }
}
This return will appear the same at the last createMember request, and now the private sensor you have added can be accessed as normal through the group using the Get Member Data request or the Get Members Data request.
