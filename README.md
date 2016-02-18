# Udacity Fullstack Nanodegree Project 4

## Description

## How to Start

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

This query will get sessions in a conference that happen on the specified date.

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

To retrieve sessions that are not workshops the != operator can be used on property *typeOfSession*. This operator is equivalent to filtering by > and < operators. That means an inequality is applied to property *typeOfSession*. To filter sessions that start before 7 PM would require another inequality operator. However, this operator would have to be applied on a different property, the *startTime* property. That is a violation of above rule.

**Solution:**
The query can retrieve either all session that are not workshops or all sessions that start before 7 PM. Either or, not both. This will results in a list that can be filtered in conference.py after retrieval. The best option would be to retrieve the fewest objects, if there's prior knowledge about either set.
