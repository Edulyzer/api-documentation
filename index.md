# External Api

Edulyzer External Api is a REST API that allows third-party systems to read data, such as answers to questionnaires, from Edulyzer.


## General

The base URL for all requests is *https://api.edulyzer.com*.

Unless otherwise noted, all requests return JSON and post requests accept JSON as a payload. 
Character encoding must be UTF-8.

The example `curl` commands below use an API Key *test*, which obviously will not work and must be replaced with a valid Key.


## Note on other APIs

There are several other APIs, as one can easily discover. Those are, however, intended for internal use only. They will work, but are subject to change and have no documentation available.

Most Edulyzer APIs follow backend-for-frontend architecture pattern, meaning that each API is made spesifically for one application.


## Swagger UI

A Swagger UI is available in the following location:

    https://api.edulyzer.com/q/swagger-ui/


## Authentication

All requests must be authenticated with an **API Key**. The API Key is included in the *Authorization* header:

```
Authorization: Bearer <API_KEY>
```

## List available schools and questionnaires

Request to */external/self* will give a list of all the schools and questionnaires the current user (API Key) has access to:

```
curl -X 'GET' \
  'https://api.edulyzer.com/external/self' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer test'
```

Will give a response like this:

```
{
  "schools": [
    {
      "id": "test-school",
      "name": "Api Key Test School",
      "role": "HEAD_MASTER"
    }
  ],
  "questionnaires": [
    "example-questionnaire"
  ]
}
```

This user has access to one school (test-school) and one questionnaire (example-questionnaire).


## Get answers for a questionnaire

Having obtained a list of available questionnaires, the */external/answers* endpoint can be used to query answers.

One first makes a post request to the */external/answers* endpoint with the following JSON payload:

```
{
  "questionnaire": "<questionnaire-id>"
}
```

That request will not yet provide any data, but will instead return a token that can later be used to get it. Loading data can take a while, so having one large blocking request would not be a good solution.

So the following example command:

```
curl -X 'POST' \
  'https://api.edulyzer.com/external/answers' \
  -H 'accept: text/plain' \
  -H 'Authorization: Bearer test' \
  -H 'Content-Type: application/json' \
  -d '{
  "questionnaire": "example-questionnaire"
}'
```

will return a token like this:

```
62512138-7e89-4d79-a9f4-8d5dd94d6226
```

Now one can issue a get request to */external/answers/{token}* endpoint to actually get the data:

```
curl -X 'GET' \
  'https://api.edulyzer.com/external/answers/62512138-7e89-4d79-a9f4-8d5dd94d6226' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer test'
```

Assuming the token is valid, this will return a JSON object that will, at least, contain a *status* object:

```
{
  "status": {
    "available": false,
    "building": true,
    "failed": false,
    "stepsCompleted": 106,
    "stepsTotal": 118
  },
  "questions": null,
  "answers": []
}
```

In case the server has not yet loaded all the data, `status.available` will be false. The fields `status.stepsCompleted` and `status.stepsTotal` can be used to monitor the progress. This case the client should wait a bit and then issue the same request with same token again. This needs to be done until `status.available` is true.

After the `status.available` is true, the response also contains the `questions` and `answers` fields. The former will list all questions in the questionnaire, including available options for each, and the latter will list the actual answers.

Note that in Edulyzer two students can see same questions at different times. This is why the `answers` array may contain questions which a student has not answered yet.

Refer to the Swagger UI for the details of the returned data.


## Providing a list of users

Edulyzer only stores the user IDs as a hash values. This means that the response from */external/answers/{token}* endpoint does not contain the real user IDs.

If the caller of this API is aware of possible user IDs, those can be provided as an input when sending the post request to the */external/answers* endpoint:

```
{
  users": [
    "user1", "user2", "user3"
  ],
  "questionnaire": "<questionnaire-id>"
}
```

Now, in case any of the hashes in Edulyzer's database match those user IDs, they will be replaced with the correct values when returning data.



## Get category data

Edulyzer comes with a lot of build-in questionnaires for various subjects. The questions from those questionnaires are grouped into categories. Each category will then have various numerical values calculated for it.

The */external/categories* endpoint can be used to query those values. This works the same way as the */external/answers* previously.

First one makes a post request to */external/categories* endpoint and sends the following JSON as a payload:

```
{
  "school": "<school-id>"
}
```

Like the */external/answers*, this endpoint will also return a token. The token can then be used to query the actual data from */external/categories/{token}* endpoint. And again, the same `status.available` field needs to be checked to see if the data is actually available.

Refer to the Swagger UI for the details of the returned data.






