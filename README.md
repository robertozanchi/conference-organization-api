# Conference Organization App API Project


## Project overview
This is project #4 of Udacity's Full Stack Web Developer Nanodegree.
The task is to add API functionalities to a WSGI web application for
conference organization. The Web Service Api is coded in [Python 2.7][1]
and hosted on [Google App Engine][2]. A fully functional basic API and
the frontend were provided by Udacity.

The API supports the following fuctionalities:
- User authentication (via Google accounts)
- User profiles, including Session wishlists
- Conference information
- Session information
- Speaker information


## Setup Instructions
1. Update the `application` value in `app.yaml` with the app ID you
   have registered in the App Engine admin console.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][3].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID.
1. Deploy your application.
1. Test API functionalities using the [Google Api Explorer][4]


## New API Functionalities

This follows the project tasks [assigned by Udacity][5].

### Task 1: Add Sessions to a Conference

To complete this task, the following functionalities were implemented:
- Session and SessionForm classes in models.py
- createSession() API endpoint
- getConferenceSessions() API endpoint
- getConferenceSessionsByType() API endpoint
- getSessionsBySpeaker() API endpoint
- Speaker and SpeakerForm classes in models.py
- addSpeaker() API endpoint
- allSpeakers() API endpoint

#### Session and SessionForm classes

Sessions are created as datastore entities with the following properties:

| Property        | Type             |
|-----------------|------------------|
| name            | string, required |
| highlights      | string           |
| speakerName     | string           |
| speakerKey      | string           |
| duration        | integer          |
| typeOfSession   | string, repeated |
| date            | date             |
| startTime       | time             |
| organizerUserId | string           |

All above values, except for `organizerUserId`, are entered using SessionForm.
Additionally, SessionForm takes the websafe key of the conference that the
session that is being created belongs to.

#### `createSession()` API endpoint

`createSession()` is an endpoint in conference.py that is used to create session
entities with the above properties in the datastore.

The logged in user's identity is matched with the conference organizer's id 
(`organizerUserId`). Only the owner of the conference may create sessions for it.

A parent-child relationship is set between the conference key and the session
key when the session key is being generated. 

#### `getConferenceSessions()` API endpoint

`getConferenceSessions()` is an ndb query that takes a websafe conference key as
input and returns all the sessions registered under it.

#### `getConferenceSessionsByType()` API endpoint

`getConferenceSessionsByType()` is an ndb query that takes a websafe conference key
and session type as inputs and returns all sessions that meet both criteria.

#### `getSessionsBySpeaker()` API endpoint

`getSessionsBySpeaker()` is an ndb query that takes a speaker's name as string as
input and returns all sessions that have that speaker, across all conferences.

#### Speaker and SpeakerForm classes

Speakers are created as datastore entities with the following properties:

| Property        | Type             |
|-----------------|------------------|
| Displayname     | string, required |
| profileKey      | string           |
| biography       | string           |

All above values are entered using SpeakerForm. profileKey is an optional value
that may be entered if speaker is also an attendee with a profile key.

Upon introducting speaker entities, I also updated `_createSessionObject()` in 
conference.py to check that the entered speaker key exists and the speaker's name.

#### `addSpeaker()` API endpoint

`addSpeaker()` is an endpoint in conference.py used to create the speaker entity with
the above properties in the datastore.

#### `allSpeakers()` API endpoint

`allSpeakers()` is an ndb query that returns all registered speakers. 

### Task 2: Add Sessions to User Wishlist

To complete this task, the following functionalities were implemented:
- `addSessionToWishlist()` API endpoint
- `getSessionsInWishlist()` API endpoint

The chosen approach for implementing session wishlists is based on making wishlists
an element of user profile entitites, where a user's wishlist is a list of session keys.

#### `addSessionToWishlist()` API endpoint

`addSessionToWishlist()` takes a websafe session key as input and appends that key to
the list of keys contained in the `sessionWishlistKeys` element within their profile.

Any user may add any conference session to their wishlist, regardless of whether they
are registered to attend the conference their desired sessions belong to or not.

#### `getSessionsInWishlist()` API endpoint

`getSessionsInWishlist()` returns a list of all the sessions saved in a user's wishlist
simply by retrieving the keys listed in the user's profile's `sessionWishlistKeys`.


### Task 3: Work on indeces and queries

To complete this task, the following functionalities were implemented:
- `sessionsMaxDuration()` API endpoint
- `sessionsStartTime()` API endpoint
- `getEarlyNonWorkshopSessions()` API endpoint (problem solution)
- Manually updated index.yaml with indices to support new endpoint queries

#### `sessionsMaxDuration()` API endpoint

`sessionsMaxDuration()` takes as input the desired maximum duration of a session 
`maxDuration` and returns all sessions with duration less or equal to that value.

#### `sessionsStartTime()` API endpoint

`sessionsStartTime()` takes a starting time `timeSTR` as input and returns all existing
sessions that commence at that exact time.

#### `getEarlyNonWorkshopSessions()` API endpoint (problem solution)

The problem: the problem presented by having to select sessions that are of a certain type
e.g. not workshops, and that commence earler than a certain time e.g. 7pm is that
potentially requires setting multiple inqualities on different fields in a ndb query.
This is not supported by Google App Engine and can return the error:
'BadRequestError: Only one inequality filter per query is supported`.

Possible solution: a solution to this problem is to only set one of the two conditions in
the ndb query in the form of an inequality, and return all sessions that match this first
condition. The results obtained are can be then filtered through iteration in Python,
where additional conditions can be applied in a for loop. `getEarlyNonWorkshopSessions()`
is an implementation of a proposed solution following this approach.

#### Updated index.yaml with indices

I manually updated the indeces in index.yaml to support all the Session queries introduces
by the new endpoints created and listed above.

### Task 4: Add a Task

To complete this task, the following functionalities were implemented:
- `cacheFeaturedSpeaker()` API endpoint
- `getFeaturedSpeaker()` API endpoint

#### `cacheFeaturedSpeaker()` API endpoint

`cacheFeaturedSpeaker()` loads a 'featured speaker' message into Memcache via `createSession()`.
In `_createSessionObject()`, before a new session is created, the speakers' name is added
as a featured speakers if the speaker has at least one other session scheduled. The speaker's
name entry into Memcache is handled using App Engine's Task Queue. 

#### `getFeaturedSpeaker()` API endpoint

`getFeaturedSpeaker()` returns the current featured speaker message loaded into Memcache. If
no speaker is featured yet, "Featured speakers to be announced soon" is set in Memcache instead.


[1]: http://python.org
[2]: https://developers.google.com/appengine
[3]: https://console.developers.google.com/
[4]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool
[5]: https://docs.google.com/document/d/1H9anIDV4QCPttiQEwpGe6MnMBx92XCOlz0B4ciD7lOs/pub#h.1xhyekfwdxa3