---
layout: post
title: "IMAPSN client specification (draft v0.3)"
---

{:toc}

# 1. Overview

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

# 2. References


[Activity Streams Concepts and Representations][json-activity.html]
: This document describes the *activity construct* and the *object
construct* and the JSON representation of these.

[activity-schema-01][activity-schema.html]
: This document describes the set of base object types and available verbs.

[JSON schema for activities][JSON schema]
: full JSON schema for activities and object constructs checked into github.

<http://json-schema.org/>
: general json schema info

<http://code.google.com/apis/gmail/oauth/>
: info and examples on using oauth

[Magic Signatures][magicsig]
: draft spec for signing javascript objects


# 3. Definitions

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


# 4. Requirements

## 4.1 Functional Requirements

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

## 4.2 Functional Flows

### 4.2.1 Use Case: make friend request and response

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

### 4.2.2 Use Case: Message an activity to a group

User's define their own groups. A special group called `everybody`
includes all accpeted friends. This list correpsonds to all messages
in the `IMAPSN` folder that have a subject that starts with
"/contacts".

1. user1 chooses a group to receive the message, and composes the message.
2. user1.client emails the `news-item` message to group
3. *-- time passes --*
4. user2.client processes `news-item` message and displays in news list

### 4.2.3 Use case: Post to a wall

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

### 4.2.4 Use Case: Reply to Activities

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

### 4.2.5 Use Case: Forward an Activity

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

### 4.2.6 Use Case: Update person data

1. user2 emails a `person-update` message containing user2's person object to user1.
2. user1 updates person data for user2.


# 5. Interfaces

## 5.1 IMAP and SMTP

All data is accessed through IMAP APIs. 
See <http://java.sun.com/products/javamail/javadocs/com/sun/mail/imap/IMAPStore.html>
or <http://docs.python.org/library/imaplib.html>.

Messages are sent using SMTP APIs.
See <http://java.sun.com/products/javamail/javadocs/com/sun/mail/smtp/package-summary.html>
or <http://docs.python.org/library/smtplib.html>


## 5.2 Data Format of E-mail messages 

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


## 5.3 Object References

It is often necessary to refer to an object for which the sender and
receiver have the full representation. In this document the term
*object reference* will be used to refer to an object in the activity
schema that has the minimum required properties to identify the
object. The two properties in an object reference are `id` and
`objectType`.


## 5.4 Message Types

The following are message types sent and processed by an IMAPSN
client.  The data property of every magic envelope contains a single
[`activity`][json-activity.html] object. The following are the types
of messages that an IMAPSN client must recognize.


### 5.4.1 `friend-request`

This is a message that includes data from a person making a request to
be a friend. Message subject is "\[IMAPSN\] friend-request:
{name of requestor}". The attached JSON data is an activity
with the following values:

    id: a unique id for the activity

    actor: `person` object of the one making the request.

    verb: http://activitystrea.ms/schema/1.0/make-friend

    object: A `person` object representing the requested friend. It
            will only include an `email` property, since the full data
            won't be known until after receiving a `friend-response`.


### 5.4.2 `friend-response`

This is a message that includes data from a person accepting a request
to be a friend.  Message subject is "\[IMAPSN\] friend-response:
{name of responder}".  The attached JSON data is an `activity`
with the following values:

    id: a unique id for the activity

    actor: The full person object of the one making the response.

    verb: http://activitystrea.ms/schema/1.0/make-friend

    object: An object reference to the `person` representing the
            friend who initiated the request.

    inReplyTo: An object reference (containing just the id) to the
               friend-request activity.

### 5.4.3 `news-item`

These messages contain an attached `activity` object that is to be
displayed in the news stream.  The message subject is 
"\[IMAPSN\] news-item: {activity.title}"


### 5.4.4 `wall-post`

A `wall-post` is an activity with a person as a target. The message
subject is "\[IMAPSN\] wall-post: {ativity.title}".

The activity contains the following properties:

    id: a unique id for the activity
    actor: object reference to the `person` making the post
    verb: any appropriate activity verb  (e.g., `tag`, `post`, `share`)
    object: the object of the verb (e.g., a `photo`)
    target: object reference to the `person` whose wall this is posted to


### 5.4.5 `direct-message`

This is a message addressed specifically to a friend or group.  The
mail subject is: "\[IMAPSN\] direct-message: {activity.title}".

The activity contains the following properties:

    id: a unique id for the activity
    actor: object reference to the `person` sending the message
    verb: post
    object: any appropriate object, typically a `note`. Could also be a `photo`, etc.


### 5.4.6 `person-update`

This is a message that replaces the current data for a friend in
the `IMAPSN` folder.  The message subject is "\[IMAPSN\] person-update:
{activity.title}" and it has an attached `person` object.


### 5.4.7 `comment`

The message subject is "\[IMAPSN\] comment: {activity.title}".
The activity has the following properties:

    id: a unique id for the activity
    actor: object reference to the `person` making the comment
    inReplyTo: the object that the comment applies to. Only includes `id`.
    verb: <http://activitystrea.ms/schema/1.0/post>
    object: a relevant object. Typically a <http://activitystrea.ms/schema/1.0/comment>.


### 5.4.8 `forward`

A forward message has a subject of "\[IMAPSN\] forward: {activity.title}".  
Its properties are:

    id: a unique id for the activity
    actor: the person sharing the comment
    verb: <http://activitystrea.ms/schema/1.0/reshare>
    annotation: a comment about the object
    object: the object that is being passed on (another activity).

This is similar to a "retweet" and gets processed as a `news-item` does.


# 6. Data Model

## 6.1 Data stored by the client

The following values are configured and stored locally by the client
for each account the client connects to. The encryption mechanism used
by the client to store passwords is not specified in this document.

<table padding="5" border="1">
  <tr><td> field</td>
      <td> type</td>
      <td> description</td></tr>
  <tr><td> account-name</td>
      <td> string(255)</td>
      <td> A name that is unique among accounts for users to identify this account.</td></tr>
  <tr><td> displayName</td>
      <td> string(255)</td>
      <td> The full name of the user that is shown to other users--"John Doe".</td></tr>
  <tr><td> email-id</td>
      <td> string(255)</td>
      <td> The email address used with this account.</td></tr>
  <tr><td> imap-host</td>
      <td> string(255)</td>
      <td> The name of the IMAP host where there is IMAPSN data.</td></tr>
  <tr><td> imap-port</td>
      <td> integer</td>
      <td> The port used when connecting to imap-host.</td></tr>
  <tr><td> imap-user</td>
      <td> string(100)</td>
      <td> The username used to log into incoming-mail-server if oauth is not supported.</td></tr>
  <tr><td> imap-password</td>
      <td> string(100)</td>
      <td> The password used to log into incoming-mail-server if oauth is not supported.</td></tr>
  <tr><td> smtp-host</td>
      <td> string(255)</td>
      <td> The name of the SMTP host used to send IMAPSN messages.</td></tr>
  <tr><td> smtp-port</td>
      <td> integer</td>
      <td> The port used when connecting to smtp-host.</td></tr>
  <tr><td> smtp-user</td>
      <td> string(100)</td>
      <td> The username used to log into outgoing-mail-server if oauth is not supported.</td></tr>
  <tr><td> smtp-password</td>
      <td> string(100)</td>
      <td> The password used to log into outgoing-mail-server if oauth is not supported.</td></tr>
  <tr><td> private-key-password</td>
      <td> string(100)</td>
      <td> The password used to decrypt the user's private key. See Section 6.5.</td></tr>
  <tr><td> imapsn-folder </td>
      <td> string(30) </td>
      <td> This is the folder on the IMAP server where all IMAPSN data is stored. 
           Defaults to IMAPSN.</td></tr>
</table> 


## 6.2 IMAP Directory layout

The data used by IMAPSN is stored in messages under an IMAP folder.
Before storing any interface messages in IMAPSN folders the magic
envelope is removed from the ``application/json`` attachment.  All
messages are stored in the configured "imapsn-folder" specified in the
client's account configuration.  The default name for this folder is
`IMAPSN`.  Each message in the `IMAPSN` folder contains an attached
JSON file.  The subject of each message is a path where each part is
separated by a "/".  The final part of the path is the name of the
attached file. Clients must not use nested IMAP Folders to organize
messages since nested Folders are not supported by all IMAP servers.

The messages in the `IMAPSN` folder have the following subjects:

         + path: /contacts/*
         |
         + path: /inbox/*
         |
         + path: /news/*
         |
         + path: /wall/*
         |
         + path: /config/*
         |
         + message: /account-owner.json
         |
         + message: /person-groups.json
         |
         + message: /person-status-map.json
         |
         + message: /key-map.json


## 6.3 Format of Data Stored on IMAP Server But Never Messaged

Each message in the `IMAPSN` folder is a Multipart/alternative MIME
email message that has the following parts:

* ``application/json``: the JSON representation serialized in plain text.
* ``text/plain``: an optional user friendly plaintext representation


## 6.4 IDs

The core object schema includes an `id` for every object. The
convention for generating globally unique id's will be to use
`acct:{email-id}#{UUID}`. Where `{email-id}` is the account email
addres and `{UUID}` is a [universally unique identifier][UUID].


## 6.5 `/account-owner.json`

`/account-owner.json` is a the subject of a message stored in the
`IMAPSN` folder.  The `application/json` attachment of this message is
an JSON object with one or more objects with the following properties.

<table padding="5" border="1"> 
   <tr><td> field </td> 
       <td> type </td>  
       <td> description </td></tr> 
   <tr><td> account-owner </td>
       <td> see [person][person.json] </td>  
       <td> the person data for this IMAPSN user. The data must have a
            "publicKey" property in the
            [magickey][application/magic-key] format, and a "keyhash"
            property that is the base64url-encoded SHA256 hash of the
            public signing key's magicsig representation. The
            person.id is formatted as: "acct:" + email_address +
            "#0"</td></tr>
   <tr><td> privateKey </td>
       <td> string </td>  
       <td> This is the encoded private key. See the next section for encoding format.</td></tr>
   <tr><td> new-message-folder </td>  
       <td> string(30) </td>  
       <td> Name of IMAP folder where new IMAPSN messages are looked
            for. Defaults to root INBOX. By configuring this value users can set up a
            mail filtering rule that puts all new IMAPSN messages a
            user-specified folder.</td></tr>
   <tr><td> wall-group </td>  
       <td> string (a group name) </td> 
       <td> The name of the group that is able to see messages posted
            to user's wall. Includes acceptance friend requests. 
            Defaults to `everybody`.</td></tr>
</table>

## 6.5.1 Private Key Storage Format

First a string is constructed with two parts "PKCS8.{data}" where
{data} is the private key serialized in PKCS#8 encoding and base64url
encoded.

Next, the key is optionally encrypted.  A key is derived from the
"private-key-password" using PBKDF2WithHmacSHA1 and the private key
string is encrypted using the derived key with the
DES/CBC/PKCS5Padding encryption scheme. See
<http://tools.ietf.org/html/rfc2898>.

Finally the three results of encryption are base64url encoded and
serialized to a string with four parts separated by a "." (0x2E)
character: 

    "DSA".<cipher text>.<salt>.<initialization vector>

The choice of whether to encrypt the key is left up to clients or
users.


## 6.6 `/person-groups.json`

A message in the `IMAPSN` folder with the subject
"/person-groups.json".  The `application/json` attachment of this
message will contain a JSON object mapping each `group-name` property
to an array of person references which contain the properties `id`,
`dislayName`, and `email`.  The `email` property is a string
containing the email address.  The `id` strings correspond to `person`
objects found in `/contacts` messages.


## 6.7 `/person-status-map.json`

A message in the `IMAPSN` folder with the subject "/person-status-map.json".
The `application/json` attachment of this message will contain a JSON
object mapping each person `id` to a status object with the folowing
properties:

<table padding="5" border="1"> 
   <tr><td> field </td> 
       <td> type </td>  
       <td> description </td></tr> 
   <tr><td> email </td>
       <td> the address of this `person` </td></tr>
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

## 6.8 `/key-map.json

The key map is a message with the subject `/key-map.json` where the
attached json data contains a map of "keyhash" values to public
keys. The key hash is the base64url-encoded SHA256 hash of the public
signing key's magicsig representation. The public key is a string
encoding of the pubic key in the [magickey][application/magic-key]
format.

## 6.9 `/contacts`

The `IMAPSN` folder will contain one message per person that is
a confirmed friend.

The subject of each message will be "/contacts/{person.id}".

The `application/json` attachment of this message is a file named
"{person.id}" that contains a [person][person.json]
object. Some of the important fields are listed here:

<table padding="5" border="1"> 
   <tr><td> field </td> 
       <td> type </td>  
       <td> description </td></tr> 
   <tr><td> email </td>
       <td> see [person][person.json] </td> 
       <td> This is where activities will be sent.</td></tr> 
   <tr><td> activity.title </td>
       <td> see [core-object.json][] </td>  
       <td> This is what is displayed as text to represnt the person.</td></tr> 
   <tr><td> image </td>
       <td> see [core-object.json][] </td>  
       <td> a media link construct referencing the avatar image displayed 
            to represent the person.</td></tr> 
   <tr><td> publicKey </td>
       <td> string </td>  
       <td> format is [magickey][application/magic-key]</td></tr> 
</table>

### 6.9.1 media link construct

The info on the media link construct is sparse in current activity stream
documentation.  Here is an example of a media link construct:

    {
        target: 'http://myvideos.com/raftingtrip/raftingvideo.avi',
        type: 'http://activitystrea.ms/schema/1.0/video',
        width: '400',
        height: '300',
        duration: '93' // in seconds
    }


## 6.10 `/news`

Messages in the `IMAPSN` folder that are news-items will have a subject
of "/news/{activity.id}".  The `application/json` attachment will be
named {activity.id} and will contain an
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


## 6.11 `/wall`

Messages in the `IMAPSN` folder that are wall-posts will have a subject
of "/wall/{activity.id}".  

## 6.12 `/inbox`

Messages in the `IMAPSN` folder that are direct messages will have a
subject of /inbox/{activity.id}". The IMAP interface is used to mark
them as read or not.


# 7. Program Control Flow

## 7.1 Client Start-up

These steps are taken every time the client starts up.

1. Process all new IMAPSN messages found in the
   "new-message-folder" conifgured in `/account-owner.json`.

2. Updates the status of each person in `/person-status-map.json`.

   1. If status is `pending` the status is left alone.
   2. if now - last-sent > 3 days set status to `neglected`.
   3. or if now - last-received > 3 days set status to `asleep`.
   4. otherwise set status to `active`.


## 7.2 Preprocess incoming email message

Before the processing for a specific message type, all incoming
messages will usuall first be processed as follows:

1. Check the `From:` address of the email message. If it is not the
   address of a confirmed friend delete the message, otherwise continue.

2. Use the email address in the `To:` field of email message to look
   up the senders's public key. 

3. Verify the signature in the magic envelope.  If the signature
   cannot be verified delete the email message, otherwise continue.

4. Look up the sender in the `/person-status-map.json` and update their
   status as prescribed in "Client Start-up".


## 7.3 Sending and Processing Message Types

The following sections give more detailed steps for the messages
described in the "Interfaces" section of this document.


### 7.3.1 Send `friend-request`

1. Construct the `friend-request` activity with a unique id, and make
   a signed magic envelope.

2. Email the `friend-request` message with the attached activity.

3. Make an initial entry in the `/person-status-map.json`.

   1. id = activity.id
   2. email = from friend request
   3. status = `pending`
   4. last-sent = now
   5. last-received = null 

The initial id value for the person-status-map entry is the the id of
the friend request activity.

### 7.3.2 Process `friend-request`

1. [Decode][decoded] the incoming `friend-request`.

2. Verify signature using the `public-key` property from the `person`
   object of the friend request. If it doesn't verify delete the
   message, otherwise save the keyhash and key in `/key-map.json` and
   continue.

3. Create an entry in the person status map:

   1. id = from actor in friend-request activity
   2. email = from actor in friend-request
   3. status = active
   4. last-sent = {current time} (when friend-response sent)
   5. last-received = {current time} (when friend-request recieved)

4. Store a message containing the decoded `person` JSON object
   representing the new friend in the `IMAPSN` folder with a subject
   of "/contacts/{activity.actor.id}".

5. Email a signed `friend-response` back to the requester. The
   activity.inReplyTo value must be the id of the incoming friend
   request.

6. Email a signed `news-item` with a
   `http://activitystrea.ms/schema/1.0/make-friend` verb to all
   friends in the configured `wall-group`.


### 7.3.3 Process a `friend-response`

1. [Decode][decoded] the incoming `friend-response` activity.

2. Verify signature using the `public-key` property from the `person`
   object of the friend response. If it doesn't verify delete the
   message, otherwise continue.

3. Update `person-status-map.json` for the responding friend

   1. Verify that a pending entry exists under the activity.inReplyTo
      id. 
   2. If it exists, remove the entry and continue, else, end.
   3. make a new entry with the actiity.actor.id and set status = `active`
   4. set last-received = {current time}

4. Store a message containing the `person` JSON object representing
   the new friend in the `IMAPSN` folder as `/contacts/{person.id}`.

5. Email a signed `news-item` with a `http://activitystrea.ms/schema/1.0/make-friend`
   to all friends in the configured `wall-group`.


### 7.3.4 Send `news-item`

1. Determine active group members in the specified group.

   1. Look up each person in the group in `/person-status-map.json`. 
   2. If status is `pending` or `asleep` then do not include that person, otherwise include.

2. Email a signed `news-item` message BCC'ed to the active group members.

3. Update the `last-sent` time for each active group member that the
   message was sent to.

4. Place a copy of sent `news-item` in the `IMAPSN` folder with a
   subject of `/news/{activity.id}`. The recipients in the `BCC:`
   header will be used when processing incoming `comment` messages.


### 7.3.5 Process a `news-item`

1. Preprocess the incoming message.

2. [Decode][decoded] the `news-item` and place a copy of sent
   `news-item` in the `IMAPSN` folder with a subject of
   `/news/{activity.id}`.

3. The news item may now be displayed in the users's news stream.


### 7.3.6 Process `wall-post`

1. Preprocess the incoming message.

2. [Decode][decoded] the `wall-post` 

   1. If the target of the activity is not the account owner place the
      wall post in the `IMAPSN` folder with a subject of
      `/news/{activity.id}" and the UI should display the item in news
      stream. End.

   2. If the target of the activity is the account owner, store the
      JSON `activity` in a message the `IMAPSN` folder with a subject
      of `/wall/{activity.id}`. Continue.

3. Determine the active group members of configured `wall-group` and
   email each a `wall-post` message with the activity attached.
   Update `last-sent`.

4. The wall post may now be displayed to the user when viewing the wall.


### 7.3.7 Send `comment`

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


### 7.3.8 Process `comment`

1. Preprocess the incoming message.

2. [Decode][decoded] the `comment` 

3. Locate the original message that the comment was made `inReplyTo`
   in the `IMAPSN` folder with a subject of "/news/{value of inReplyTo}".

4. If the original message was not created by `account-owner` the
   comment will be displayed by a UI in the news stream with the
   original. End.

5. Otherwise, if the original message was created by `account-owner`
   find list of recipients the original message was sent to.

4. Sign and email the comment to the original recipients. 

5. Update the `last-sent` time for each recipient.

6. The comment may now be displayed to the user.


### 7.3.9 `forward` an activity

This is done just like a `news-item` except the activity should be
constructed as described previously for a `forward` message.

Once an activity is forwarded, comments are sent to and distributed by
the person who forwarded the message.


### 7.3.10 Process a `forward` activity

This is processed just like a `news-item`.


### 7.3.11 Send `direct-message`

1. Determine list of recipients (may be a group or may be directly
   adressed).  Note: mesasges are sent regardless of status in
   `person-status-map.json` being `pending` or `asleep`.

2. Email a signed `direct-message` message to the recipients using the
   `BCC:` header.

3. Update the `last-sent` time in `person-status-map.json` for each recipient.

4. Place a copy of sent `direct-message` in the `IMAPSN` folder with a
   subject of "/inbox/{activity.id}".


### 7.3.12 Process incoming `direct-message`

1. Preprocess the incoming `direct-message`.

2. [Decode][decoded] the `direct-message` and store the JSON
   `activity` in a `direct-message` within the `IMAPSN` folder with a
   subject of "/inbox/{activity.id}".

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

