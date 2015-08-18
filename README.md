# skhonference

This is the final project for Udacity's course

"Developing Scalable Apps in Python"

This solution is based on the sample code provided by Udacity in 

  https://github.com/udacity/ud858

See README_UPSTREAM.md for instructions how to setup and deploy the application.

I have documented my test cases in textplan.txt

Task 1: Add Sessions to a Conference

I have modeled speakers as separate entities with the fields

- name
- bio (optional)

Speakers do not have ancestors.
Speaker names do not need to be unique - names in real life are not unique, either.

When creating sessions, the speaker can be given by name or websafe key. If the speaker name is given, but two speakers of this name exist, an error is returned and the websafe key must be used.

The value of the field 'typeOfSession' is restricted to the values defined in the Enum 'sessionType':

- WORKSHOP
- LECTURE
- TUTORIAL
- BOF
- UNSPECIFIED

Default is 'UNSPECIFIED'.

When a Session is created, the Conference is set as its ancestor. This relationship is used in the query getConferenceSessions().

Sessions do not need to have a speaker. There might be speaker-less session types, or the conference organizer does not know the speaker yet, but wants to reserve a slot for the session.

Sessions can't have more than one speaker.

I am using the following forms for creation and display of speakers and sessions:

- speakerForm: outbound, all fields in the model plus websafe key
- speakerMiniForm: inbound, only editable fields
- speakerForms: outbound, a list of speakerForm items as query answer

- sessionType: inbound/outbound Enum for the values of 'typeOfSession'
- sessionForm: outbound, all fields in the model plus websafe key
- sessionMiniForm: inbound, only editable fields
- sessionForms: outbound, a list of sessionForm items as query answer

I use the following URL paths and HTTP methods for the API methods handling speakers and sessions:

- /speaker, POST, createSpeaker()
- /speakers, GET, getSpeakers()
- /session/{websafeConferenceKey}, POST, createSession()
- /sessions/{websafeConferenceKey}, GET, getConferenceSessions()
- /sessions_by_type/{websafeConferenceKey}, POST, getConferenceSessionsByType()
- /sessions_by_speaker, POST, getSessionsBySpeaker()

I use GET wherever possible and pass the websafeConferenceKey as part of the URL path when it is the ancestor key. If I need to send more parameters, either for object creation of for a query, I use POST.


Task 2: Add Sessions to User Wishlist

A user may add any session to their wishlist, regardless whether they have registered for the conference or not.

The wishlist is kept in the Profile model in a newly added property:

  sessionWishlist = ndb.StringProperty(repeated=True)

It contains a list of websafe keys of Session objects.

I use the following ResourceContainers for wishlist management:

WISHLIST_REQUEST (for DELETE and POST methods)
CONF_WISHLIST_GET_REQUEST (for the GET method where a URL path parameter is needed)

Both contain only a VoidMessage as first parameter and define the parameter used in the URL paths below.

I use the following URL paths and HTTP methods for the wishlist API methods:

- /wishlist/{websafeSessionKey}, POST, addSessionToWishlist()
- /wishlist/{websafeSessionKey}, DELETE, removeSessionFromWishlist()
- /wishlist, GET, getWishlistSessions()
- /wishlist, GET, getSessionsInWishlist()

I use POST to add a session to the wish list, and DELETE to remove it from it.

getWishlistSessions() lists all sessions in the wishlist across all conferences, and getSessionsInWishlist() gets all sessions in the wishlist in a specific conference. (I would have called the latter getConferenceSessionsInWishlist() to be consistent with other conference-related methods, but the method name was explicitly given in the task.)


Task 3: Work on indexes and queries

Part 1: Indexes

The following index had to be added to support the queriy needed for the added functionality in part 3 of this task:

- kind: Session
  properties:
  - name: __key__
  - name: startTime

I did not have to add indexes to support queries from tasks 1 and 2.

Part 2: Additional queries

I have added a query to list all topics that were given to conferences in the system, and a query to list all conferences for one given topic. In a UI I would use the first query to build a dropdown menu in a search form, which would then as input for the second query.

For this feature I have created the following forms:

- TopicForm: inbound/outbound, containing just the topic string
- TopicForms: outbound, a list of TopicForm items as query answer

I use the following URL paths and HTTP methods for the topic search API methods:

- /topics, 'GET', getTopics()
- /conferencesbytopic, 'POST', getConferencesByTopic()


Part 3: Query problem

Google App Engine Data Store does not allow queries for more than one inequality for performance reasons. I can either search for sessions NOT matching a certain session type ("!=" is an inquality), or for sessions EARLIER than a given time ("<" is the other inequality).

One solution is to create two queries, and use the result set from one query as filter for the other, with the IN filter operator (which checks if an item is part of a given list).

For even more complex queries another solution would be to loop over the result of one query and decide with complex if conditions whether to copy each item to a manually created result set.

For this task I have chosen the first solution and implemented it in the following API method:

- /sessionquery/{websafeConferenceKey}, POST, getSessionsBeforeExcluding()

The query gets the websafe key for the conference as an URL parameter, and the session type to exclude as well as the cut-off time as POST parameters. I have create the form

- SessionQueryAfterExcludingForm

and the ResourceContainer

- SESSIONS_BEFORE_EXCLUDING_POST_REQUEST

as supporting data structures for this query.





