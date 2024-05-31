# Goals
- Allow client app to use response `data` from CRUD operations to sync changes to its local database.
- To minimize network traffic, networking data usage while maintaining consistency & performance.

# Endpoint Naming Convention & Response Structure

### [C]reate a Resource
`POST` `{{service}}/api/{{version}}/{{resources}}`  
> Status: 201 (Created)  
> Response:`data` is the newly created resource  

E.g.  
`POST` `feed/api/v1/feeds`
```
{
  "status": 201,
  "message": "Successful",
  "data": {
    "feedID": "fc724e5018f94bf49d094d498af043a4",
    ...
  }
}
```

---
### [R]ead Resource (Listing)
`GET` `{{service}}/api/{{version}}/{{resources}}`  
`GET` `{{service}}/api/{{version}}/{{parent-resources}}/{{parent-resource-id}}/{{resource-id}}`  
> Status: 200 (OK)  
> Response: `data` is an array of resource  

E.g.  
`GET` `feed/api/v1/feeds/fc724e5018f94bf49d094d498af043a4/reactions`
```
{
  "status": 200,
  "message": "Successful",
  "data": [
    {
      "reactionID": "fc724e5018f94bf49d094d498af043a4",
      ...
    }
  ],
  "paging": { }
}
```

---
### [R]ead Resource with Toggle-able Actions (Listing) 
`actions` is needed for client app to show selection state. E.g. reaction with type `HAHA`, client app will show Haha as selection on react button. We will need a all properties (excluding parent resource id, `userID`, and `creationTime`)  
Resources with toggle-able actions are `feeds`, `comments`, `people_profile`

`GET` `{{service}}/api/{{version}}/{{resources}}`  
> Status: 200 (OK)  
> Response: `data` is an array of resource  

E.g.  
`GET` `feed/api/v1/feeds`
```
{
  "status": 200,
  "message": "Successful",
  "data": [
    {
      "feedID": "fc724e5018f94bf49d094d498af043a4",
      ...
      "actions": {
        "reaction": {
          "reactionID": "",
          "typeID": "HAHA",
          ...
        }
      }
    }
  ],
  "paging": { }
}
```

`GET` `feed/api/v1/feeds/fc724e5018f94bf49d094d498af043a4/comments`
```
{
  "status": 200,
  "message": "Successful",
  "data": [
    {
      "commentID": "fc724e5018f94bf49d094d498af043a4",
      ...
      "actions": {
        "reaction": {
          "reactionID": "",
          "typeID": "HAHA",
          ...
        }
      }
    }
  ],
  "paging": { }
}
```

`GET` `people/api/v1/users/fc724e5018f94bf49d094d498af043a4`
```
{
  "status": 200,
  "message": "Successful",
  "data": {
    "userID": "fc724e5018f94bf49d094d498af043a4",
    ...
    "actions": {
      "follow": {
        "followID": "",
        "targetUserID": "fc724e5018f94bf49d094d498af043a4",
        ...
      }
    }
  }
}
```

---
### [R]ead Resource (Detail)
`GET` `{{service}}/api/{{version}}/{{resources}}/{{resource-id}}`  
> Status: 200 (OK)  
> Response: `data` is the resource detail  

E.g.  
`GET` `feed/api/v1/feeds/fc724e5018f94bf49d094d498af043a4`  
```
{
  "status": 200,
  "message": "Successful",
  "data": {
    "feedID": "fc724e5018f94bf49d094d498af043a4",
    ...
  }
}
```

---
### [U]pdate a Resource
`PATCH` `{{service}}/api/{{version}}/{{resources}}`  

---
### [D]elete Resource
`DELETE` `{{service}}/api/{{version}}/{{resources}}/{{resource-id}}`  
> Status: 204 (No Content)  
> Response: none  

E.g.  
`DELETE` `feed/api/v1/feeds/fc724e5018f94bf49d094d498af043a4`  

---
### Perform Action on Resource (statistics-affected)
`POST` `{{service}}/api/{{version}}/{{resources}}/{{resource-id}}/{{action}}`  

Status is 200 (OK). Response (`data` is the newly created action resource, `statistics` is for affected parent resource)  
  
E.g.  
`POST` `feed/api/v1/feeds/fc724e5018f94bf49d094d498af043a4/react`  
```
{
  "status": 200,
  "message": "Successful",
  "data": {
    "reactionID": "",
    ...
  },
  "statistics": {
    "reactions": 5,
    "comments": 5,
    ...
  }
}
```

`POST` `feed/api/v1/feeds/fc724e5018f94bf49d094d498af043a4/comment`  
```
{
  "status": 200,
  "message": "Successful",
  "data": {
    "commentID": "",
    ...
  },
  "statistics": {
    "reactions": 5,
    "comments": 5
    ...
  }
}
```

`POST` `feed/api/v1/comments/fc724e5018f94bf49d094d498af043a4/comment` a.k.a reply (recursive parent)  
```
{
  "status": 200,
  "message": "Successful",
  "data": {
    "commentID": "",
    ...
  },
  "statistics": {
    "reactions": 5,
    "comments": 5
    ...
  }
}
```

`POST` `people/api/v1/users/fc724e5018f94bf49d094d498af043a4/follow`  
```
{
  "status": 200,
  "message": "Successful",
  "data": {
    "followID": "",
    ...
  },
  "statistics": {
    "following": 10,
    "followers": 10,
    ...
  }
}
```

---
### Delete Action Resources by Parent Resource (statistics-affected)
This is used for toggle-able actions such as reaction (feed), save (feed), archive (feed), follow (people), etc.  

`DELETE` `{{service}}/api/{{version}}/{{resources}}/{{resource-id}}/{{action-resources}}`
> Status: 200 (OK)  
> Response: `data` is an array of deleted action resources - only `ID`  

E.g.  
`DELETE` `feed/api/v1/feeds/fc724e5018f94bf49d094d498af043a4/reactions`  
```
{
  "status": 200,
  "message": "Successful",
  "data": [
    {
      "reactionID": ""
    }
  ],
  "statistics": {
    "reactions": 5,
    "comments": 5,
    ...
  }
}
```
`DELETE` `people/api/v1/users/fc724e5018f94bf49d094d498af043a4/follows`  
```
{
  "status": 200,
  "message": "Successful",
  "data": [
    {
      "followID": ""
    }
  ],
  "statistics": {
    "following": 10,
    "followers": 10,
    ...
  }
}
```

# Actions
Feed actions: `react` (toggle), `comment`, `share`, `save` (toggle), `archive` (toggle), ~~`report`~~

Comment actions: `react` (toggle), `comment` (as reply), ~~`report`~~

People actions: `follow` (toggle), ~~`see_first` (toggle)~~, ~~`block` (toggle)~~, ~~`report`~~

# Statistics (Public)
Feed statistics: `reactions`, `comments`, `shares`, `saves`

Story statistics: `views`

Comment statistics: `reactions`, `comments`

People profile statistics: `following`, `followers`


# Feed & Share
Feed types: `POST`, `SHARE`, `REEL`, ~~`LIVE`~~, ~~`AD`~~   
Share types: `POST`, `REEL`, ~~`LIVE`~~, ~~`AD`~~   

# Deep Links (Universal)
- https://`{{web_url}}`/`{{user_id}}` redirect to `{{username}}` variant below  
- https://`{{web_url}}`/`{{username}}`  
- https://`{{web_url}}`/`{{username}}`/posts/`{{post_id}}`   
- https://`{{web_url}}`/`{{username}}`/stories/`{{story_id}}`  
- https://`{{web_url}}`/`{{username}}`/reels/`{{reel_id}}`  
- https://`{{web_url}}`/`{{username}}`/lives/`{{live_id}}`  

`web_url` will be kiloapp.com, uat.kiloapp.com, dev.kiloapp.com
