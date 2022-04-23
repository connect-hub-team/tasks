# Tasks and plans repo
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
// note: message data can contain anything from plain text 
// to an image reference (images are stored in File storage)
// note: messages are only deleted in secret chats. They are marked as 
// deleted otherwise. This is done to slow down database defragmentation 
// and improve performance.
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
// note: we do not store users in room data because it may cause 
// a huge performance hit when a room has many users
{
  "user_id": ...,
  "chat_id": ...,
  "data": {
    "is_muted": ...,
  }
}
```

### Content store
Stores files and associated information such as upload date and owner. Actual files are stored on the disk and database is used for storing the metadata.
```json
// FILE
// note: when forwarding a message a new file record referencing 
// the old file will be created. This way if sender deletes 
// the message file will still be accessible on the forwarded one
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

## Logic

### Content store

The server has little storage space so additional nodes are used to store files

Problems with nodes
- Node might not have a static ip address
- Node might have a slow connection

Solution 
- Use ssh to proxy connections to nodes
- Cache frequently used files on the server

#### File download
When downloading a file server first check the cache. If file is present, it is send back. Otherwise request is forwarded to a node that has the file.

![File download](img/contentstore%20take%201-page1.png)

#### File upload
All uploaded files are stored in cache at first. A file can be moved to a node if server is running out of space.

![File upload](img/contentstore%20take%201.png)