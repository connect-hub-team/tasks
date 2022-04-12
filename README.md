# Tasks repo
Project plans and project structure goes here. Everything described below is a subject to change.
## Database Structure
This section describes to database looks like. Right now we are using Postgres. Although we might use MongoDB or Redis for some cases in the future.
### Auth database
Auth database stores common user data such as credentials and user settings in the future.
```json
// USER
{
  "id": ...,
  "email": ...,
  "pass_hash": ...,
}

// USER SETTINGS
{
  todo
}

// implemented in second round
// SESSION
{
  "id": ...,
  "ip": ...,
  "user_agent": ...,
  "data": {
    ...
  }
}
```

### Chat database
Communications such as messages and calls are stored in this database.
```json
// ROOM
{
  "id": ...,
  "name": ...,
  "data": {
    "roles": [
        {
          "id": ...,
          "name": ...,
          "permissions": ...,
        }
      ],
    "permissions": ...
  },
  "type": enum // e.g. dialog, group
}
// MESSAGE
// note: message data can contain anything from plain text to an image reference (images are stored in File storage)
// note: messages are only deleted in secret chats. They are marked as deleted otherwise. This is done to slow down database defragmentation and improve performance.
{
  "id": ...,
  "room_id": ...,
  "data": {
    "text": ...
    "seen": ...
    ""
  },
  "sent_at": ...,
  "type": enum // e.g. message, event, user joined event
  "status": enum // e.g. sent, deleted, edited
}
// USER : ROOM (many to many)
// note: we do not store users in room data because it may cause a huge performance hit when a room has many users
{
  "user_id": ...,
  "chat_id": ...,
  "data": {
    "is_muted": ...,
  }
}
```

### File storage
Stores files and associated information such as upload date and owner. Actual files are stored on the disk and database is used for storing the metadata.
```json
// FILE
// note: when forwarding a message a new file record referencing the old file will be created. This way if sender deletes the message file will still be accessible on the forwarded one
{
  "id": ...,
  "owner": ...,
  "filename_visible": ...,
  "filename_internal": ...,
  "uploaded_at": ...,
  "size": ...,
  "hash": ...,
  "type": ...,
}

// FILE PERMISSIONS
{
  "file_id": ...,
  "data": {
      ...
  },
}
```