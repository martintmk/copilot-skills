---
name: teams-self-message
description: >
  Send a text message to the current user's Microsoft Teams self-chat. Use when
  the user says "message myself", "send me a Teams message", "send this to my
  Teams", or invokes this skill with message text. The message is posted as the
  user to the Teams "Just me" chat, so Teams may not produce an incoming-message
  notification.
---

# Teams Self Message

Send the user's supplied text once to their Microsoft Teams self-chat.

## Input

Treat the text accompanying the invocation as the message body. Preserve its
wording, punctuation, and line breaks. Remove only an invocation wrapper such as
`/teams-self-message` when it is not part of the intended message.

If no message text was supplied, use `ask_user` to request it. Do not invent a
message.

## Procedure

1. Ensure the WorkIQ `create_entity` operation is available. If WorkIQ tools are
   deferred, discover the create operation with the tool-search mechanism before
   calling it. If authentication is requested, let the user complete the
   configured WorkIQ OAuth flow.
2. Create exactly one entity at:

   `/me/chats/48:notes/messages`

3. Use this JSON body, substituting the supplied text for `<message>`:

   ```json
   {
     "body": {
       "contentType": "text",
       "content": "<message>"
     }
   }
   ```

   Do not add `@odata.type` fields. The Teams self-chat endpoint rejects the
   item-body type emitted by that payload shape.
4. Treat HTTP status `201` as success. Report that the message was sent.

## Failure handling

- Do not retry after a success or an ambiguous transport failure: a retry could
  duplicate the message.
- For an explicit failure response, report the service error concisely and do
  not claim the message was sent.
- Do not substitute email, a normal one-to-one chat, a channel post, or a bot
  notification.
- Never include credentials, access tokens, tenant identifiers, or raw response
  metadata in the final response.
