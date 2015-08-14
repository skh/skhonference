# skhonference

Based on https://github.com/udacity/ud858

Design considerations

Sessions:
- typeOfSession is a field with a fixed set of possible values, these values
  are defined in sessionTypes, an Enum

- highlights -- possibly no to be indexed, thus TextProperty
- abstract -- not in the spec, but a session needs some kind of description
  Indexed (by default, and not turned off.)

Sessions are children of conferences, so Conference needs a session field too

No default values in sessions, only name mandatory

Forms needed:
- sessionForm with full information also available in the model, plus a websafe key
- sessionEditForm with all information that can be edited by the creator
- sessionForms, a list of sessionForm as query answer


endpoints to be implemented
- createSession(SessionForm, websafeConferenceKey)
- getSessionsBy

