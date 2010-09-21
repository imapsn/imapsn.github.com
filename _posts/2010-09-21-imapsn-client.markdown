---
layout: post
title: "IMAPSN client specification"
---

{:toc}

Overview
========

The goal of IMAPSN is to create a social networking platform

*   that isn't run by a single company 
*   that provides a choice of revenue models (advertising, paid subscription)
*   where the users control their content and interactions
*   that is easy to set up
*   that a lot of people will use

There is already a platform in wide use that might fit the
bill. Everyone already has email. There are existing choices in
providers. People who care a little more about privacy can use paid
services. Others can use advertising funded services. Implementing
some of the social networking concepts on email will not require
complicated deployment of anything new since the infrastructure
already exists.

While an IMAPSN client could be implemented as a MUA (think thunderbird
plugin), a web application platform and a browser interface is also a
viable choice, and the goal of this specification is to define enough
of the functionality so that a variety of clients can inter-operate.
Then software developers can begin experimenting with creating their
own user interfaces and approaches to attracting users. It's important
to note that, from the user perspective, IMAPSN is nothing like
email. The user interface should look more like Facebook or Myspace
than an email inbox.

Email as a storage and messaging platform has two serious drawbacks:

* scaling into friend-of-friend territory will cause email traffic to
  grow very quickly and needs limits (namely comment visibility has to
  stay in the friend circle of the original message)

* messages are pushed instead of pulled, which is a waste because not
  every status update from every friend will be read.

Assuming we can address these problems or live with some trade-offs,
here is a design for implementing social networking concepts on email.

References
==========

[Activity Streams Concepts and Representations][json-activity.html]
: This document describes the *activity construct* and the *object
construct* and the JSON representation of these.

[activity-schema-01][activity-schema.html]
: This document describes the set of base object types and available verbs.

[JSON schema for activities][JSON schema]
: full JSON schema for activities and object constructs checked into github.

<http://json-schema.org/>
: general json schema info

[beginner's guide to oauth](http://hueniverse.com/2007/10/beginners-guide-to-oauth-part-i-overview/)
: overview of oauth

[Magic Signatures][magicsig]
: draft spec for signing javascript objects

Definitions
===========

*account*
: a single email address or identity for which an IMAPSN client sends and receives messages.
From the perspective of IMAPSN, all services (IMAP, SMPT) and their
configuration information for a single email address form an account.

*[activity][]*
: "In its simplest form, an activity consists of an actor, a verb, and
an object. It tells the story of a person performing an action on or
with an object -- 'Geraldine posted a photo' or 'John shared a
video'. In most cases these components will be explicitly declared,
but they may also be implied." -- [Activity Streams Concepts and
Representations][json-activity.html].

*[comment][]*
: The "comment" object type represents a textual response to another
object.

*friend*
: a reciprocal relationship between two users. user1 requests to make
user2 a friend and user2 confirms. once the relationship is
established the two users can send and receive activities as email
messages.

*group*
: a set of friends

*[status][]*
: The "status" Object type represents a human-readable update of the
author's situation, mood, location or other status.

Requirements
============

## Functional Requirements

IMAPSN is built on an IMAP server and SMTP server.

An IMAPSN client to sends [activities][json-activity.html] to friends
using email.  Activities received from friends are accessed on the
IMAP server, processed, and displayed by the IMAPSN client. 

The IMAPSN client will do the following:

*   Process a *make friend* request
*   Send a *make friend* request
*   Retrieve and display a list of friends
*   Display the details of a particular friend
*   Create groups (add and remove friends from groups)
*   Message an activity to a group (share status update, link, photo, make friend event, etc.)
*   Discard incoming messages that are not from a confirmed friend
*   Retrieve and display a list of activities
*   Allow the user to *comment on* and *like* activities
*   Allow the user to forward activities
*   Display the details of a particular activity
*   Message an incoming comment back out to the group to which the activity was originally sent
*   Add an account
*   Switch between accounts

## Functional Flows

### Use Case 1: make friend request and response

1. user1.client emails a `friend-request` message to user2 
2. *-- time passes --*
3. user2.client displays friend request to user2, user2 confirms
4. user2.client emails a `friend-response` message to
   user1 using the email address in the `friend-request` message.
5. user2.client emails an `activity` message with the `make-friend`
   verb to friends in the configured `wall-group`. (The activity only
   includes the new friend's name, email address, and image.)
6. *-- time passes --*
7. user1.client opens new `friend-response` message on user1.imap
8. user1.client emails an activity with the `make-friend` verb to `wall-group`.
9. user1.client notifies user1 of friendship.

### Use Case 2: Message an activity to a group

User's define their own groups. A special group called `everybody`
includes all persons in `IMAPSN/contacts`. Messages are always sent
to groups using BCC.

1. user1 chooses a group to receive the message, and composes the message.
2. user1.client emails the `news-item` message to group
3. *-- time passes --*
4. user2.client processes `news-item` message and displays in news list

### Use case 3: Post to a wall

1. user1.client emails a `wall-post` message to a confirmed friend, user2.
2. *-- time passes --*
3. user2.client processes new message containing `wall-post`.
4. user2.client emails `wall-post` message to configured `wall-group`.
5. *-- time passes --*
6. userN.client (userN is a friend of user2) displays data from all
   `wall-post` messages received from user2 when userN chooses
   to view user2's wall.

A variation of this is tagging multiple people in an image. The image
is posted to the wall of each person tagged.

### Use Case 4: Reply to Activities

This is the functional flow for user2 replying to ("liking" or
"commenting" on) an activity received from user1.

1. user2 composes a reply or clicks "like" in response to an `activity` received from user1.
2. user2.client emails a `comment` message to user1.
3. *-- time passes --*
4. user1.client processes new `comment` message.
5. user1.client emails `comment` message to all persons in the
   group the original message was emailed to.
6. *-- time passes --*
7. userN.client (userN is a friend of user1 who received the original
   message) processes the incoming `comment` message, displaying it
   with the original.

By design, the replies are restricted to the persons included in the
original message. Friends of user2 will not see user2's replies made
to user1, unless they are also friends of user1 and received the
original message.  user2 has the option of forwarding an activity
with a comment. Then the replies will be limited to the group user2
forwarded to, which may or may not include user1.

### Use Case 5: Forward an Activity

1. user2 receives `news-item-1` from user1.
2. user2 wants to share `news-item-1` with some of her own freinds
3. user2.client emails a `forward` message with an optional comment to a selected group.
   userN is a friend of user2 in the messaged group
4. *-- time passes --*
5. userN.client processes new `forward` message and displays the
   forwarded news item in userN's news stream.

It is possible for a user to have one of their items forwarded back to
them. The forwarded item will collect it's own set of replies
separately from the original.  User interfaces will have to convey
that any comments on the forwarded item involve a different set of
people making replies than the original and the people commenting on
the forwarded item will not necessarily see the comments made on the
original.  The recommended approach is to make the forward a visually
separate item in the news stream, with an indication of who forwarded
the item.

### Use Case 6: Update person data

1. user2 emails a `person-update` message containing user2's person object to user1.
2. user1 updates person data for user2.


Interfaces
==========

## IMAP and SMTP

All data is accessed through IMAP APIs. 
See <http://java.sun.com/products/javamail/javadocs/com/sun/mail/imap/IMAPStore.html>
or <http://docs.python.org/library/imaplib.html>.

Messages are sent using SMTP APIs.
See <http://java.sun.com/products/javamail/javadocs/com/sun/mail/smtp/package-summary.html>
or <http://docs.python.org/library/smtplib.html>


## Data Format of E-mail messages 

Each message will have a subject of "\[IMAPSN\] {message type} : {activity.title}".

When addressing a message the `To:` address must match the `From:` address
and all recipients must be included as `BCC:`.

Each message is a Multipart/alternative MIME email message that has
the following parts:

* ``application/json``: the JSON representation of an activity serialized as a [magic envelope][].
* ``text/plain``: a user friendly plaintext representation
* ``text/html``: an optional user friendly html representation

An `X-IMAPSN-object-id` email header must be present and have as its
value the `id` property of the encoded activity attached to the
message.


## Object References

It is often necessary to refer to an object for which the sender and
receiver have the full representation. In this document the term
*object reference* will be used to refer to an object in the activity
schema that has the minimum required properties to identify the
object. The two properties in an object reference are `id` and
`objectType`.


## Message Types

The following are message types sent and processed by an IMAPSN
client.  The data property of every magic envelope contains a single
[`activity`][json-activity.html] object. The following are the types
of messages that an IMAPSN client must recognize.


### `friend-request`

This is a message that includes data from a person making a request to
be a friend. Message subject is "\[IMAPSN\] friend-request:
{name of requestor}". The attached JSON data is an activity
with the following values:

    actor: `person` object of the one making the request.

    verb: http://activitystrea.ms/schema/1.0/make-friend

    object: A `person` object representing the requested friend. It
            will only include an `email` property, since the full data
            won't be known until after receiving a `friend-response`.


### `friend-response`

This is a message that includes data from a person accepting a request
to be a friend.  Message subject is "\[IMAPSN\] friend-response:
{name of responder}".  The attached JSON data is an `activity`
with the following values:

    actor: The full person object of the one making the response.

    verb: http://activitystrea.ms/schema/1.0/make-friend

    object: An object reference to the `person` representing the
            friend who initiated the request.


### `news-item`

These messages contain an attached `activity` object that is to be
displayed in the news stream.  The message subject is 
"\[IMAPSN\] news-item: {activity.title}"


### `wall-post`

A `wall-post` is an activity with a person as a target. The message
subject is "\[IMAPSN\] wall-post: {ativity.title}".

The activity contains the following properties:

    actor: object reference to the `person` making the post
    verb: any appropriate activity verb  (e.g., `tag`, `post`, `share`)
    object: the object of the verb (e.g., a `photo`)
    target: object reference to the `person` whose wall this is posted to


### `direct-message`

This is a message addressed specifically to a friend or group.  The
mail subject is: "\[IMAPSN\] direct-message: {activity.title}".

The activity contains the following properties:

    actor: object reference to the `person` sending the message
    verb: post
    object: any appropriate object, typically a `note`. Could also be a `photo`, etc.


### `person-update`

This is a message that replaces the current data for a friend in
IMAPSN/contacts.  The message subject is "\[IMAPSN\] person-update:
{activity.title}" and it has an attached `person` object.


### `comment`

The message subject is "\[IMAPSN\] comment: {activity.title}".
The activity has the following properties:

    actor: object reference to the `person` making the comment
    inReplyTo: the object that the comment applies to. Only includes `id`.
    verb: <http://activitystrea.ms/schema/1.0/post>
    object: a relevant object. Typically a <http://activitystrea.ms/schema/1.0/comment>.


### `forward`

A forward message has a subject of "\[IMAPSN\] forward: {activity.title}".  
Its properties are:

    actor: the person sharing the comment
    verb: <http://activitystrea.ms/schema/1.0/share>
    object: the object that is being passed on.

This is similar to a "retweet" and gets processed as a `news-item` does.


Data Model
==========

## Data stored by the client

The following values are configured and stored locally by the client
for each account the client connects to

<table padding="5" border="1">
  <tr><td> field</td>
      <td> type</td>
      <td> description</td></tr>
  <tr><td> incoming-mail-server</td>
      <td> string(255)</td>
      <td> The name of the IMAP host where there is IMAPSN data.</td></tr>
  <tr><td> incoming-username</td>
      <td> string(100)</td>
      <td> The username used to log into incoming-mail-server if oauth is not supported.</td></tr>
  <tr><td> incoming-password</td>
      <td> string(100)</td>
      <td> The password used to log into incoming-mail-server if oauth is not supported.</td></tr>
  <tr><td> outgoing-mail-server</td>
      <td> string(255)</td>
      <td> The name of the SMTP host used to send IMAPSN messages.</td></tr>
  <tr><td> incoming-username</td>
      <td> string(100)</td>
      <td> The username used to log into outgoing-mail-server if oauth is not supported.</td></tr>
  <tr><td> incoming-password</td>
      <td> string(100)</td>
      <td> The password used to log into outgoing-mail-server if oauth is not supported.</td></tr>
  <tr><td> sftp-server</td>
      <td> string(255)</td>
      <td> Optional. web server host for uploads. This is to support
           uploading and sharing images and other media. Images will
           not be attached to emails. </td></tr> 
  <tr><td> sftp-username </td> 
      <td> string(100) </td> 
      <td> The username used to log into sftp-server. </td></tr> 
  <tr><td> sftp-password </td> 
      <td> string(100) </td> 
      <td> The password used to log into sftp-server. </td></tr> 
  <tr><td> IMAPSN-folder </td>
      <td> string(30) </td>
      <td> This is the folder on the IMAP server where all IMAPSN data is stored. 
           Defaults to IMAPSN.</td></tr>
</table> 


## IMAP Directory layout

The data used by IMAPSN is stored in messages under an IMAP folder.
Before storing any interface messages in IMAPSN folders the magic
envelope is removed from the ``application/json`` attachment.  The
default directory layout is

    + folder: IMAPSN
    |
    +----+ message: config-data
         |
         + folder: contacts
         |
         + message: person-status-map
         |
         + message: person-groups
         |
         + folder: news
         |
         + folder: wall
         |
         + folder: inbox


## Format of Data Stored on IMAP Server But Never Messaged

Since the data will be stored in email messages each message will have
a standard format.  The subject will be "\[IMAPSN\] {message type} : {activity.title}".

Each message is a Multipart/alternative MIME email message that has
the following parts:

* ``application/json``: the JSON representation serialized in plain text.
* ``text/plain``: an optional user friendly plaintext representation


## IDs

The core object schema includes an `id` for every object. The
convention for generating globally unique id's will be to use
`http://{hostname from email address of account-owner}/{[UUID][]}`. 


## `IMAPSN/config-data`

`config-data` is a the subject of a message stored in the IMAPSN
folder.  The `application/json` attachment of this message is an array
with one or more objects with the following properties.

<table padding="5" border="1"> 
   <tr><td> field </td> 
       <td> type </td>  
       <td> description </td></tr> 
   <tr><td> account-name </td>
       <td> string </td>  
       <td> the user visible name of the account. </td></tr> 
   <tr><td> account-owner </td>
       <td> see [person.json][] </td>  
       <td> the person data for this IMAPSN user. </td></tr> 
   <tr><td> private-key </td>
       <td> string </td>  
       <td> format is tbd. </td></tr> 
   <tr><td> new-message-folder </td>  
       <td> string(30) </td>  
       <td> Name of IMAP folder where new IMAPSN messages are looked
            for. Defaults to root INBOX. By configuring this value users can set up a
            mail filtering rule that puts all new IMAPSN messages a
            user-specified folder.</td></tr>
   <tr><td> auto-approve-follow-requests </td>  
       <td> boolean </td> 
       <td> If false follow requests must be confirmed by the user,
            otherwise they are approved automatically. </td></tr>
   <tr><td> wall-group </td>  
       <td> string (a group name) </td> 
       <td> The name of the group that is able to see messages posted
            to user's wall. Includes acceptance friend requests. 
            Defaults to `everybody`.</td></tr>
</table>


## `IMAPSN/contacts`

The IMAPSN/contacts folder will contain one message per person that is
a confirmed friend.

The subject of each message will be "\[IMAPSN\] person: {activity.title}".

The `application/json` attachment of this message is a
[person][person.json] object. Some of the important fields are listed
here:

<table padding="5" border="1"> 
   <tr><td> field </td> 
       <td> type </td>  
       <td> description </td></tr> 
   <tr><td> email </td>
       <td> see [person.json][] </td> 
       <td> This is where activities will be sent.</td></tr> 
   <tr><td> activity.title </td>
       <td> see [core-object.json][] </td>  
       <td> This is what is displayed as text to represnt the person.</td></tr> 
   <tr><td> image </td>
       <td> see [core-object.json][] </td>  
       <td> a media link construct referencing the avatar image displayed 
            to represent the person.</td></tr> 
   <tr><td> public-key </td>
       <td> string </td>  
       <td> format is [magickey][application/magic-key]</td></tr> 
</table>

### media link construct

The info on the media link construct is sparse in current activity stream
documentation.  Here is an example of a media link construct:

    {
        target: 'http://myvideos.com/raftingtrip/raftingvideo.avi',
        type: 'http://activitystrea.ms/schema/1.0/video',
        width: '400',
        height: '300',
        duration: '93' // in seconds
    }

## `IMAPSN/person-status-map`

A message in the IMAPSN folder with the subject "\[IMAPSN\]
person-status-map".  The `application/json` attachment of this message
will contain an array of objects with the folowing properties:

<table padding="5" border="1"> 
   <tr><td> field </td> 
       <td> type </td>  
       <td> description </td></tr> 
   <tr><td> email </td>
       <td> email address of a person in IMAPSN/contacts </td>  
       <td>  </td></tr>
   <tr><td> internal-id </td>
       <td> string </td>
       <td> a uuid-id generated by the client to identify 
            the person associated with this email address </td></tr>
   <tr><td> status </td>
       <td> string </td>  
       <td> one of `pending`, `asleep`, `neglected`,  or `active`.</td></tr> 
   <tr><td> last-sent </td>
       <td> ISO 8601 date-time string  </td>  
       <td> Time when an activity or message was last sent to this person. </td></tr> 
   <tr><td> last-received </td>
       <td> ISO 8601 date-time string  </td>  
       <td> Time when an activity or message was last received from this person. </td></tr> 
</table>

## `IMAPSN/person-groups`

A message in the `IMAPSN` folder with the subject "\[IMAPSN\]
person-groups".  The `application/json` attachment of this message
will contain an array of objects that each contain a `group-name`
property and a `persons` property that is an array of `id`
strings that correpsond to `person` objects in `IMAPSN/contacts`.


## `IMAPSN/news`

Messages in the IMAPSN/news folder will have a subject of
"\[IMAPSN\] activity: {activity.title}".  The `application/json`
attachment of this message will contain an
[activity][activity.json]. Some of the important fields are listed
here.

<table padding="5" border="1"> 
   <tr><td> field </td> 
       <td> type </td>  
       <td> description </td></tr> 
   <tr><td> actor </td>
       <td> object </td>  
       <td> this will be minimum data, that is, the id, since we have the full object. </td></tr> 
   <tr><td> verb </td>
       <td> URI </td>  
       <td> For example `http://activitystrea.ms/schema/1.0/share` </td></tr> 
   <tr><td> object </td>
       <td> object </td>  
       <td> The primary object of the activity. Example: a photo.</td></tr> 
</table>


## `IMAPSN/wall`

This is the folder where processed incoming `wall-post` messages are
stored. A message is removed from the user's wall by deleting it from
this folder.

## `IMAPSN/inbox`

This is the folder where processed incoming `message` messages are
stored. The IMAP interface is used to mark them as read or not.





Program Control Flow
====================

## Client Start-up

These steps are taken every time the client starts up.

1. Process all new IMAPSN messages found in either the IMPAP INBOX
   folder of in IMAPSN/contacts folder based on their `Subject:`
   fields.

2. Updates the status of each person in IMAPSN/contacts.

   1. If status is `pending` the status is left alone.
   2. if last-received - last-sent > 3 days set status to `neglected`.
   3. if last-sent - last-received > 3 days set status to `asleep`.
   4. otherwise set status to `active`.


## Preprocess incoming email message

Before the processing for a specific message type, all incoming
messages will usuall first be processed as follows:

1. Check the `From:` address of the email message. If it is not the
   address of a confirmed friend delete the message, otherwise continue.

2. Use the email address in the `To:` field of email message to look
   up the senders's public key. 

3. Verify the signature in the magic envelope.  If the signature
   cannot be verified delete the email message, otherwise continue.

4. Look up the sender in the `person-status-map` and update their
   status as prescribed in "Client Start-up".


## Sending and Processing Message Types

The following sections give more detailed steps for the messages
described in the "Interfaces" section of this document.


### Send `friend-request`

1. Construct the `friend-request` activity, and make a signed magic
   envelope.

2. Email the `friend-request` message.

3. Make an initial entry in the `person-status-map`.

   1. id = null
   2. email = from friend request
   3. status = `pending`
   4. last-sent = now
   5. last-received = null 


### Process `friend-request`

1. [Decode][decoded] the incoming `friend-request`.

2. Verify signature using the `public-key` property from the `person`
   object of the friend request. If it doesn't verify delete the
   message, otherwise continue.

3. Create an entry in the person status map:

   1. internal-id = id from `friend-request`
   2. email = from friend-request
   3. status = `active`
   4. last-sent = {current time} (when friend-response sent)
   5. last-received = {current time} (when friend-request recieved)

4. Store a message containing the decoded `person` JSON object
   representing the new friend in `IMAPSN/contacts`.

5. Email a signed `friend-response` back to the requester.

6. Email a signed `news-item` with a
   `http://activitystrea.ms/schema/1.0/make-friend` verb to all
   friends in the configured `wall-group`.


### Process a `friend-response`

1. [Decode][decoded] the incoming `friend-response` .

2. Verify signature using the `public-key` property from the `person`
   object of the friend response. If it doesn't verify delete the
   message, otherwise continue.

3. Update `person-status-map` for the responding friend

   1. status = `active`
   2. last-received = {current time}

4. Store a message containing the `person` JSON object representing
   the new friend in `IMAPSN/contacts`.

5. Email a signed `news-item` with a `http://activitystrea.ms/schema/1.0/make-friend`
   to all friends in the configured `wall-group`.


### Send `news-item`

1. Determine active group members in the specified group.

   1. Look up each person in the group in `person-status-map`. 
   2. If status is `pending` or `asleep` then do not include that person, otherwise include.

2. Email a signed `news-item` message BCC'ed to the active group members.

3. Update the `last-sent` time for each active group member that the
   message was sent to.

4. Place a copy of sent `news-item` in `IMAPSN/news`. The recipients in
   the `BCC:` header will be used when processing incoming `comment`
   messages.


### Process a `news-item`

1. Preprocess the incoming message.

2. [Decode][decoded] the `news-item` and store the JSON `activity` in
   a message within the `IMAPSN/news` folder.

3. The news item may now be displayed in the users's news stream.


### Process `wall-post`

1. Preprocess the incoming message.

2. [Decode][decoded] the `wall-post` 

   1. If the target of the activity is not the account owner place the
      wall post in the IMAPSN/news folder and display in news stream. End.

   2. If the target of the activity is the account owner, store the
      JSON `activity` in a `wall-post` message within the `IMAPSN/wall`
      folder. Continue.

3. Determine the active group members of configured `wall-group` 
   and email each the `wall-post`.  Update `last-sent`.

4. The wall post may now be displayed to the user when viewing the wall.


### Send `comment`

1. Construct an activity and set inReplyTo to an object with an `id`
   that references the original. 

2. Email a signed `comment` message to author of the original activity
   at the root of the inReplyTo chain. The root is the first object
   found when traversing `inReplyTo` properties that doesn't
   have an `inReplyTo` property.

   A `comment` may be made on other comments so the structure will be
   nested, and the email must be sent to the root person who initiated
   the discussion. This is important because the distribution of
   comments is handled by the original author.

3. Update the `last-sent` time for every person in the inReplyTo chain
   who is a friend.


### Process `comment`

1. Preprocess the incoming message.

2. [Decode][decoded] the `comment` 

3. Locate the original message that the comment was made `inReplyTo`
   in `IMAPSN/news`

4. If the original message was not created by `account-owner` display
   it in news stream with the original.

5. Otherwise, if the original message was created by `account-owner`
   find list of recipients the original message was sent to.

4. Sign and email the comment to the original recipients. 

5. Update the `last-sent` time for each recipient.

6. The comment may now be displayed to the user.


### `forward` an activity

This is done just like a `news-item` except the activity should be
constructed as described previously for a `forward` message.

Once an activity is forwarded, comments are sent to and distributed by
the person who forwarded the message.


### Process a `forward` activity

This is processed just like a `news-item`.


### Send `direct-message`

1. Determine list of recipients (may be a group or may be directly
   adressed).  Note: mesasges are sent regardless of status in
   `person-status-map` being `pending` or `asleep`.

2. Email a signed `direct-message` message to the recipients using the
   `BCC:` header.

3. Update the `last-sent` time in `person-status-map` for each recipient.

4. Place a copy of sent `direct-message` in `IMAPSN/inbox`.


### Process incoming `direct-message`

1. Preprocess the incoming `direct-message`.

2. [Decode][decoded] the `direct-message` and store the JSON `activity` in
   a `direct-message` within the `IMAPSN/inbox` folder.

3. The new message may now be displayed in the users's message inbox.


[json-activity.html]: http://activitystrea.ms/head/json-activity.html
[activity]: http://activitystrea.ms/head/json-activity.html#json.activity


[activity-schema.html]: http://activitystrea.ms/head/activity-schema.html
[person]: http://activitystrea.ms/head/activity-schema.html#person
[comment]: http://activitystrea.ms/head/activity-schema.html#comment
[status]: http://activitystrea.ms/head/activity-schema.html#status

[horse]: http://www.dugnorth.com/blog/uploaded_images/steel_horse-764912.jpg

[JSON schema]: http://github.com/activitystreams/json-schema
[person.json]: http://github.com/activitystreams/json-schema/blob/master/objectTypes/person.json
[core_object.json]: http://github.com/activitystreams/json-schema/blob/master/core_object.json
[activity.json]: http://github.com/activitystreams/json-schema/blob/master/activity.json

[UUID]: http://en.wikipedia.org/wiki/Universally_unique_identifier

[magicsig]: http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-00.html
[magickey]: http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-00.html#anchor11
[magic envelope]: http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-00.html#anchor5

[decoded]: http://salmon-protocol.googlecode.com/svn/trunk/draft-panzer-magicsig-00.html#encoding_details

