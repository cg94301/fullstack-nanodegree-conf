# Udacity Fullstack Nanodegree Project 4

## Description

This project extends the Conference Central application from Udacity. This application allows to create, search and register for conferences. 

You can get the Udacity stater code at this URL:

`https://github.com/udacity/ud858`

Currently the Conference Central application is pretty limited - conferences have just name, description and date when the conference happens. But usually conferences have more than that - there are different sessions, with different speakers, maybe some of them happening in parallel! Your task in this project is to add this functionality. Some of the functionality will have well defined requirements, some of it will more open ended and will require for you think about the best way to design it.

You do not have to do any work on the frontend part of the app to successfully finish this project. All your added functionality will be testable via APIs Explorer.

You will have to submit your app ID, and text for the parts of the project that require explanation.




## How to Start

Fork the project from `https://github.com/cg94301/fullstack-nanodegree-conf`. Then clone the project you just forked.

This project uses Google App Engine. You need to download and install App Engine for Python. You'll also need to create an App Engine web client project and choose your app ID. Google will provide a web client ID. Then replace the given App ID and web client ID with yours in these files:

```
  conference.py: WEB_CLIENT_ID = 'your_web_client_id'
  app.yaml: application: 'your_app_id'
  static/js/app.js: CLIENT_ID: 'your_web_client_id'
```

Then start the application using the App Engine development server:

```
  $ dev_appserver.py fullstack-nanodegree-conf
  INFO     2016-02-18 19:33:10,538 api_server.py:205] Starting API server at: http://localhost:55880
  INFO     2016-02-18 19:33:10,542 dispatcher.py:197] Starting module "default" running at: http://localhost:8080
  INFO     2016-02-18 19:33:10,544 admin_server.py:116] Starting admin server at: http://localhost:8000
```

You need to use Firefox to work with the API. Open Firefox and goto `localhost:8080`. Use the UI to log into the application. Then go to your profile and select a T-shirt size. Then create a conference via UI. You will need the webSafeConferenceKey that will be created for this conference later. You can find this key by going to the maintenance URL at `localhost:8000`. Select Datastore Viewer to find the key.


## TASK 1: Add Sessions to a Conference

The following endpoints methods have been defined:

`getConferenceSessions(websafeConferenceKey)` -- Given a conference, return all sessions

`getConferenceSessionsByType(websafeConferenceKey, typeOfSession)` -- Given a conference, return all sessions of a specified type (eg lecture, keynote, workshop)

`getSessionsBySpeaker(speaker)` -- Given a speaker, return all sessions given by this particular speaker, across all conferences

`createSession(SessionForm, websafeConferenceKey)` -- open only to the organizer of the conference


## TASK 2: Add Sessions to User Wishlist

The wishlist has been added as property `sessionKeysToAttend = ndb.StringProperty(repeated=True)` to the `Profile` class in models.py.

The following endpoints methods have been define:

`addSessionToWishlist(SessionKey)` -- adds the session to the user's list of sessions they are interested in attending

`getSessionsInWishlist()` -- query for all the sessions in a conference that the user is interested in

`deleteSessionInWishlist(SessionKey)` -- removes the session from the user’s list of sessions they are interested in attending


## TASK 3

### 3.1 Create indexes

Run on localhost first to auto-generate indices in index.yaml, e.g.:

```
- kind: Session
  ancestor: yes
  properties:
  - name: duration
```

### 3.2 Come up with 2 additional queries

This query will get session in a conference that are equal to or shorter than specified duration.

```
SESSION_GET_REQUEST_BYDURATION = endpoints.ResourceContainer(
    duration = messages.StringField(1),
    websafeConferenceKey = messages.StringField(2),
)

    @endpoints.method(SESSION_GET_REQUEST_BYDURATION, SessionForms,
            path='getConferenceSessionsByDuration',
            http_method='POST', name='getConferenceSessionsByDuration')
    def getConferenceSessionsByDuration(self, request):
        """Get sessions that are shorter or equal to duration."""
        return self._getSessions(request)

    def _getSessions(self, request):
           
          ...

          # if duration has been specified, filter by that
          if hasattr(request, 'duration') and request.duration:
            print request.duration
            sessions = sessions.filter(Session.duration <= int(request.duration))
            sessions = sessions.filter(Session.duration > 0)
```

This query will get sessions in a conference that happen on the specified date. The date is entered as string, in this format: `2016-02-27`

```
SESSION_GET_REQUEST_BYDATE = endpoints.ResourceContainer(
    date = messages.StringField(1),
    websafeConferenceKey = messages.StringField(2),
)

    @endpoints.method(SESSION_GET_REQUEST_BYDATE, SessionForms,
            path='getConferenceSessionsByDate',
            http_method='POST', name='getConferenceSessionsByDate')
    def getConferenceSessionsByDate(self, request):
        """Get sessions that happen on date."""
        return self._getSessions(request)

    def _getSessions(self, request):
         ...

         # if date has been specified, filter by that
          if hasattr(request, 'date') and request.date:
            print request.date
            sessions = sessions.filter(Session.date == datetime.strptime(request.date[:10], "%Y-%m-%d").date())
```

### 3.3 Solve the following query related problem

**Query:**
> Let's say that you don't like workshops and you don't like sessions after 7 pm. How would you handle a query for all non-workshop sessions before 7 pm? What is the problem for implementing this query? What ways to solve it did you think of?

**Problem:**
The problem with this query is that it violates the following rule: An inequality filter can be applied to at most one property.

To retrieve sessions that are not workshops the `!=` operator can be used on property `typeOfSession`. This operator is equivalent to filtering by `>` and `<` operators. That means an inequality is applied to property `typeOfSession`. To filter sessions that start before 7 PM would require another inequality operator. However, this operator would have to be applied on a different property, the `startTime` property. That is a violation of above rule.

**Solution:**
The query can retrieve either all session that are not workshops or all sessions that start before 7 PM. Either or, not both. This will results in a list that can be filtered by a method in conference.py after retrieval. The best option would be to retrieve the fewest objects, if there's prior knowledge about either set.

## TASK 4

When a new session is added to a conference, the speaker is checked. If there is more than one session by this speaker at this conference, a new Memcache entry that features the speaker and session names is added. The Memcache key is `MULTI_SESSIONS`. The logic above is handled by using App Engine's Task Queue.

The following endpoints method has been implemented:

`getFeaturedSpeaker()`