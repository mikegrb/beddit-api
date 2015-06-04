# User account related resources

## POST /api/v1/user/password_reset

Request password reset for user. The user will be sent an email which includes
instructions for setting a new password.

**Authentication**

Request must be authenticated by Application specific token.

**Request**

```javascript
{
  "email" : "example@email.com"
}
```

**Response**


```javascript
{
  "email" : "example@email.com"
}
```

**Errors**

Error identifier | HTTP Status | Description
-----------------|-------------|------------
no_such_user | 400 | User account with that email was not found.
email_send_failed | 500 | Password reset email could not be sent. This can happen due to invalid email address, or temporary problem with email sending service.


## GET /api/v1/user/:user_id

Return the current user profile data.

### Authentication

Request must be authenticated by User specific token.

### Response

```javascript
{
  "id" : 123,
  "email" : "example@email.com",
  "name" : "Mikko W",
  "date_of_birth" : "1981-03-05",
  "sex" : "male",
  "weight" : 65.5,
  "height" : 169.0,
  "sleep_time_goal" : 28800,
  "tip_audiences" : ["general"],
  "created" : 1371472503.646541,
  "updated" : 1371492826.623422
}
```

If the field value is not set, it is **null**.

### Fields

Fields that belong to the sleep object.

Field | Description
---------|--------
id | Unique and permanent user id.
email | User's email address. **Must not be null**
name | User's name, as she want's to be called.
date_of_birth | User's date of birth.
sex | Biological sex. Choises are "male", "female", and null.
weight | Weight in kilograms
height | Height in centimeters
sleep_time_goal | Sleep time goal in seconds. **Must not be null**
tip_audiences | Sleep tip categories the user has chosen to see. May be empty list, but not null.
created | Timestamp of user creation
updated | Timestamp of user profile update


## PUT /api/v1/user/:user_id

Update all or some of the fields in user profile.

### Request

The data to update is sent in request body in JSON format, as in the GET
request. Id, created, and updated fields are ignored, if present.

Fields can be removed (cleared) by setting a field value to null in the request.


### Response

Returns the updated user profile, see GET request.


# Group resources

Groups provide a way to form peer groups. Members of a group can all view their
sleep and other data. Members can also invite new members to group or remove
members from the group.

## GET /api/v1/user/:user_id/group	

Return a list of groups the user belongs to.

**Authentication**

The request requires authentication with the specified user's access token. You
can only list your own groups.

**Example response**

```javascript
[
  {
    "id" : 1234,
    "created" : 1371472503.646541,
    "members" : [
      <USER_PROFILE_1>,
      ...
    ],
    "pending_invites" : [
      {
        "created" : 1371472503.646541,
        "created_by" : 132, // Beddit user id
        "email" : "example@beddit.com",
        },
        ...
     ]
    },
    ...
]
```

Field | Description
------|------------
id | Unique identifier of the group
members | List of group member's user profile (see documentation for that)
pending_invites | List of invitations that are not yet been replied to

## POST /api/v1/group/:group_id/invite
## POST /api/v1/group/new/invite

Invite a new member to group. Creating an invitation sends an invite email to
specified Beddit user. The email address must belong to an existing Beddit user
account.

A member can be invited to existing group, or a new one.

**Authentication**

The request must be authenticated with a group member's access token.

**Request body example**

```javascript
{
  "email" : "example.user@beddit.com"
}
```

Field | Description
------|------------
email | Beddit account email address of the invited user. The account must already exist.

**Response**

If everything went ok, returns the updated group listing (same as returned by
/api/v1/user/:user_id/group). The client application can then update it's
internal group model.

**Errors**

HTTP status | Code            | Description
------------|-----------------|------------
400         | email_not_found | Beddit account with given email does not exist.
400         | already_member  | The user is already member of the group.


## GET /api/v1/group/:group_id/invite/:invite_code/accept

Accept invite to a group.

If the specified group and invite code are not valid, 404 is returned.

Note that more appropriate HTTP method would be POST, because the request
creates a new relationship between the accepting user and the group. However,
as the method is also designed to work from within invite email, and support
for making POST requests from email clients is limited, GET method is used as
a workaround. 

**Response**

Returns a HTTP redirect to Beddit family app.


## POST /api/v1/group/:group_id/member/:user_id/remove

Remove a member from the group.

When a group only has one member left, the group object is deleted.

**Authentication**

The request must be authenticated with a group member's access token.

**Response**

If everything went ok, returns the updated group listing.
