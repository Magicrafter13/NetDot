# NetDot Network Protocol Revision 3 (Draft)
This is going to be a working document detailing a proposed implementation for a new net-code specification. I'm open to feedback, whether it be what you consider a better way of doing something, or simply proposing a feature. The [history](#history) section of the document will detail why I am doing this, as well as attempt to document [protocol 2](#20-net-code), and the [original protocol](#1x-net-code).

## Stipulations
We must establish some baseline facts about the NetDot game, and the server client model.
- This is a multiplayer (networked) game of "dots n' boxes".
- It follows a strict server/client model. The server is authoritative, and clients are expected to follow its command.
- The clients may keep track of as much or as little information as they want (so long as it does not interfere with the smooth operation of the game).
- Servers listen on a port (default 1234), with a basic TCP socket, and thread off connections.

## Specification
Honestly, I still like the idea of categorizing commands (the `a-b` format). Though I may have to consider replacing the hyphen with a space for simplicity (and we could allow for `a` commands that don't have any `b`s). I do realize that some commands may make it hard to program things for the same reason I want to make an API. Because of this commands will have to be introduced to designate the start and end of a list or sequence of commands. That way a client can store the data as it comes in, and only when signaled, commit that new data (UI update).

First thing's first, we have to get connected. I believe the queue/lobby/spectator/player system is still a good idea. The handshake will continue to be `info-version`, because the more I think about it, the more I enjoy the idea of making a 3-version compatible server (not making it a requirement for the specification of course). The standard shall be that the server sends this information first, but this will not be a requirement, so that older clients can be convinced to stay connected if compatibility is desired.

When first connected, the "queue" as it was once called, will simply be the default state for any connected client. It will allow simple communication with the server to gain any info the client wishes to know. This will be known as the talking state (subject to change). The primary use of this would likely be a server browser, as with a direct connection one would already be trying to join the game.

When the client requests info about the server, more info should be sent than just the version, which will hopefully become more clear as I write more. But for now, it needs to inform the client of whether or not voting is enabled on the server. (Anything like this in the future, if I forget, someone please tell me. Optional server features should be communicated when possible.)

If a client decides (or has already decided) they wish to join this server, the server must do one of three things. The server may deny this request for any reason, and also close the connection if desired. If the server is full (or doesn't want someone to join the game but is willing to let them spectate), it must assign the user to spectator mode. Or, the server may assign the user a unique player ID, allowing them into the game - the game is not required to be stopped (though how one handles that would be up to them).

Players should be allowed to go into spectator mode while the game is running. This will be treated similarly to a network disconnect in 2.0, but without requiring the user to actually disconnect. They may simply decide to not continue playing this particular game.

When the server is in the lobby state, players may freely switch in/out of spectator mode (so long as the player limit, if set, isn't exceeded). The server may shuffle the player order while in the lobby, if desired, to allow for less repetitive gameplay. The server will remain in the lobby until it decides to begin a game, unless [voting](#voting) is enabled.

When a game is running, in contrast to previous protocols, the server will send out authoritative commands (like the original spectator commands), and clients will update their game, without processing the move for validation. This means server will also send out box updates. While this puts a tiny bit more effort on the server's end, it will help avoid bad situations in the future, and also could allow for some creative uses of a server, perhaps to make art.

### States
Table showing what states a command group is acceptible in. This is not specifically referring to servers or clients, and is rather generalized.
| Group | Talk/Queue | Lobby | Spectating | Playing | Notes |
| ----- | ---------- | ----- | ---------- | ------- | ----- |
| info  | Required   | No    | No         | No      |       |
| feature | Required | Yes   | Yes        | Yes     | For now this whall work in all modes. I like the idea of a server being able to disable/enable features when it feels like it. Perhaps the people are getting a little too rowdy so the server operator disables voting? |
| vote  | No         | Yes   | Yes        | Yes     | See [voting](#voting). |
| network | Yes*     | Yes   | Yes        | Yes     | *Should only be used to assign the user an ID |
| request | Yes      | Yes   | Yes        | Yes     | Both for requestion information, and for things to happen? |

### Voting
A new voting system shall be added. A server may choose whether or not to allow voting, however this should be communicated to the client when they first connect.

For ease of use, only one vote shall run at any given time. It will be initiated by `vote-start`, and ended with `vote-end`. When starting a vote, the command should be followed by some word that indicates what is being voted on. It may be any officially documented voting category, or a custom one, as clients are not required to respond to votes, so unsupported/unknown votes can just be ignored by a client. For example, if someone wants to restart a game, they would send `vote-request restart` to the server, and the server would send out `vote-start restart`. The responses you must support will be `vote-yes` and `vote-no`. But custom responses may be allowed (however, once again, a client need not support them).

There should also be `vote-enable <feature: String>` and `vote-disable <feature: String>` to vote to enable/disable a server feature. Obviously these are `yes`/`no` responses.

- `restart`: Used for new games from the lobby and restarting a running game (context sensitive). `yes`/`no`
- `lobby`: Stop a game and return to the lobby. `yes`/`no`
- `shuffle`: Shuffle player order (should we finish one player cycle first?). `yes`/`no`

### Official/Custom Features
These are features that servers should likely support, but at minimum be prepared to handle commands that involve them even if they don't implement the feature. I.e. a server may choose to not support the randomized starting player feature, but must still be able to communicate that to all clients as it is considered an official feature.

- `random-start`: When play begins, the server will randomly decide who gets to play first, instead of defaulting to player 0 (likely the server).
- `vote`: Whether or not the server supports/allows voting.
- `chat`: Although chat communication can be rather important, some servers may wish to have silent games. I would recommend that if you don't support chat, that you do support voting, so people can at least communicate their intention to restart games.

A server may implement custom features as long as their name/string does not conflict with an existing feature. Because of this I suppose clients should also communicate what features they support so a server may decide whether or not to allow them in. Because, not supporting a feature doesn't necessitate kicking a player, unless that feature is crucial to the game.

### Command Groups
- `info`: All things information sharing. Version numbers, server capacity, motd, etc. Any type of query you might wish to make on a server, even if you aren't trying to connect to it to play.
- `feature`: Specialized information sharing - that at present is allowed during every state - which informs clients of which of the optional features mentioned in this specification, are enabled/disabled.
- `vote`: Used for initiating, and participating in, server votes. See [voting](#voting).
- `network`: Updates regarding the state of the network.
- `request`: For clients, this is to request information, and for the server to do things. For servers this is just for requesting information

## Server Requirements

## Client Requirements
This information is relevant to anyone wishing to create a client following this protocol. To ensure user experience (and to avoid silly server requirements), these are not optional unfortunately.
- Keep score. There are many things the server can do for the client, but score is fundamental to this game. There are also many things you can [leave out](#client-flexibility), so I have to make this a minimum requirement. Although the server could inform you when a player has won, there are many instances where some information may change, that would warrant a UI update. Asking the server to tell you the current winner every time a change like that happens is stupid. So for you to know the winner, you must know player scores.

## Client Flexibility
This information is relevant to anyone wishing to create a client following this protocol, and details things that are not required, but one may wish to add for the best experience.
- Color: The server will store and send out color information for every player. This is used to color lines, boxes, and possibly player names in chat. This used to be a pencil and paper game, so obviously that isn't required.
- While traditionally this information does not matter, and will not change the game, lines are assigned (and colored) to the player that made them. Clients may choose to ignore this information, and only track who a box belongs to, but this is less visually appealing.

# History
Below documents my original implementations of the network protocol for NetDot. Every command has an intention but those intentions were never required to be followed (which I abused, for example, when I added a feature to the Android version allowing the game to randomly pick the starting player - this worked by sending out `game-current` which was intended for spectators).

This is a large reason why I am deciding to draft up a 3.0 revision. Although I find it cool that I could add a feature by re-using an existing command in a way it was never intended, I realize this makes creating a game server/client somewhat ambiguous. Although I never said one could not send `game-current` to players at any point in the game, it stands to reason someone that knew that it was intended for spectators only, might program their clients to ignore such commands when they are part of the game, thus breaking the "feature" I created.

This kinda goes against the point of my version number system in the first place - while also not doing that... All servers/clients with version 1.x should be compatible, and all with 2.x should be compatible. The minor version number is supposed to indicate non-breaking differences. Like when chat was added. This wouldn't break existing clients or servers, they would simply inform the sender that they didn't know how to handle the command and leave it at that.

All that being said, take the command descriptions below as referring to my intentions, but not required for compatibility.

## 2.0 Net-code
While not completely dissatisfied with this protocol, I've already stated my reason for desiring change. Surprisingly never got past 2.0, though a version 2.1 did exist for one commit.

Version 2.0 (while not necessarily tied to the network protocol), introduces essentially 3 possible states for a client to be in. Queued (connected to the server, nothing else). Spectating (watching the game but not actively participating). And Player/Playing... which should be self explanatory. Before, spectating was not a thing, and the queue was just a technicality (see `network-busy` in 1.x). The clients are expected to be somewhat aware of this paradigm.

The server only has two states, lobby and game. This is basically how 1.0 worked, but it is more solidified here. In the lobby no game actions can be taken, and players can leave and join freely (unless the server is full), and in-game acts as it did before, though allowing for clients to join as spectators.

### Shared commands
- `network-disconnect`: No change from 1.0, except for clients: this no longer indicates the removal of some player, this now works the same way for the client as it did for the server, except it's even harsher. You *are* leaving the server, goodbye.
- `info-version <major: Int> <minor: Int>`: No change from 1.0.
- `info-malformed`: No change from 1.0, except clients can send this now too, not just servers.
- `unknown` or `unknown-`: No change from 1.0.
- `unknown-<something>`: No change from 1.0.

### Server to Client
- `player-add <id: Int> <name: String>`: Essentially a combination of `network-connect` and `player-new` in 1.0, but with the score argument dropped, as this should only be sent while in the lobby. (Players can't join during games so everyone should know the score already!)
- `player-rename <id: Int> <name: String>`: No change from 1.0.
- `player-color <id: Int> <color: Int>`: No change from 1.0.
- `player-remove <id: Int>`: Replaces `network-disconnect <id: Int>` from 1.0. Although it does still imply someone disconnected, its purpose is to remove a player from the game, hence the change. It was also for consistency/organization, as all `player` commands are for modifying data about players!
- `player-line <id: Int> <x: Int> <y: Int> ("hor" or "ver")`: This is identical to `game-play` with the same arguments, except it is intended for spectators when giving them all the game data at once. The reason these two things exist is down to a strange legacy decision. Clients may take instructions from the server, but my client code is written in such a way that it runs the valid move check even though it should just take the server's word as fully authoritative. This will definitely be changed in revision 3, but there's nothing stopping a server from only using spectator commands like this to make sure no client bugs affect their ability to play.
- `player-box <id: Int> <x: Int> <y: Int>`: Similar intention to `player-line`. This is to tell spectators about already captured boxes when they join. Only dot coordinates are needed of course.
- `grid-size <width: Int>x<height: Int>`: No change from 1.0.
- `grid-reset`: No change from 1.0.
- `network-assign <id: Int>`: No change from 1.0. This moves a player from the queue or spectators list, into the game. Although I suppose its possible a spectator might want to remain spectating without playing... there's something for the revision!
- `network-busy`: Like 1.0 this indicates a game is in progress, except now it's not just there to crush your day, you can send `request-spectate`!
- `network-chat <id: Int> <message: String>`: Identical to 1.0, except, the `id` can now be negative. These negative values have special meaning. -1 I don't believe was ever used in chat, but internally it represents a queued client - I honestly can't remember if you need to support this or not, I'd think queued clients wouldn't be allowed to chat. -2 represents a spectator. -3 represents a special message from the server and should be printed as is (or maybe with a special prefix denoting it is a server message) - I used this when clients pressed the start game button for example, the server would send out `network-chat -3 xxx wants to start the game!`.
- `network-full`: Sent after attempting to join the game, when the server reached its player limit. May be treated like `network-busy` but does not indicate a game in progress, as the server could be in lobby mode.
- `game-start`: Is now identical to `game-restart`.
- `game-restart`: No change from 1.0.
- `game-stop`: No change from 1.0, now explicitly signals the server is returning to lobby mode.
- `game-current`: Intended for newly joined spectators, to tell them whose turn it is (which clients are normally expected to keep track of themselves). I abused this to add a neat feature to the Android version. See the [History](#history) section.
- `info-warn`: No change from 1.0.
- `request-deny`: Probably should have done `info-deny`, since the server itself is not making a request, but this command indicates the server denied your recent request (`join`, `spectate`, etc).

### Client to Server
- `player-rename <name: String>`: No change from 1.0.
- `player-color <color: Int>`: No change from 1.0.
- `network-disconnect`: No change from 1.0.
- `network-chat <message: String>`: No change from 1.2.
- `game-play <x: Int> <y: Int> ("hor" or "ver")`: Functionally identical to 1.0, but swaps coordinate pair and verticality, removes the comma, and changes a `Boolean.toString` (ew) with solidified strings for each case, `hor` for horizontal (right) and `ver` for vertical (down).
- `request-start`: Is now the same as `request-restart`.
- `request-restart`: No change from 1.0.
- `request-stop`: No change from 1.0, now explicitly used to request the server return to the lobby state.
- `request-join`: No longer used to annoy users while a game is going (since spectator mode exists), this is now (supposed to be) a required command you must issue to leave the queued state and become a player. (Will be denied if server is full or game has started.)
- `request-spectate`: If you can't join the game (or don't wish to...), you must send this command to leave the queue, and to actually be able to see the game and see/participate in chat.

## 1.x Net-code
Version 2 certainly improved things, but the 1.x protocol isn't *unusable*. However if you wanted an ultimate compatible server that worked with 1, 2, and whatever comes in the future, you'd be quite limited with these clients...

### Commit Timeline
(From the original CSE 223 repo.)
- 1.0: `bb50e366`
- 1.1: No formal documentation of its existence...
- 1.2: Mentioned in 1.3 (added `network-chat`)
- 1.3: `f3cf06e8` - not a true update to the protocol, only to the Java UI!
- 1.4: `36fd1ab6` - once again, no change to the protocol
- 1.5: `f705527e` - no change to protocol...
- 1.6: `42524351` - okay I think `network-chat` was the only real protocol change at this point
- 1.7: `3a73fd02` - admittedly a huge rewrite, but still, no, protocol change!

### Shared commands
- `info-version <major: Int> <minor: Int>`: The sender is informing you of what version of the network protocol it supports. You are expected to disconnect if the major version number is different to your own, but the sender will probably disconnect you when you send your version anyway... A clever coder could implement a compatibility layer in their server to support multiple versions potentially.
- `unknown` or `unknown-` (I think): The receiver did not recognize the command group of some command you sent. For example, in `player-rename`, `player` is the command group.
- `unknown-<something>`: The receiver did not recognize the command you sent, from the `something` group (but it did recognize the group).

### Server to Client
- `player-count <count: Int>`: The server is telling you how many players are in the game. It appears this was not something to send during a game, as my implementation clears the player list when it receives this and creates a new one of the correct size.
- `player-new <id: Int> <score: Int> <name: String>`: Add a new player to the game, with the given id, score, and name. Or that's what I thought until I saw `network-connect`. This command is actually to set info for a player, but they should already be in your player list... wow.
- `player-rename <id: Int> <name: String>`: Change the name of the player with the given id, to the given name.
- `player-color <id: Int> <color: Int>`: Change the color of the player with the given id, to the given color.
- `grid-size <width: Int>x<height: Int>`: The game grid is now this size, with width and height referring to number of dots in each direction (so they should be greater than 1 to be valid, since 1x1 has no boxes).
- `grid-reset`: Reset your grid data. (Clear owned lines/boxes, resize if necessary, whatever.)
- `network-connect`: Apparently this is what is sent when a new player joins the game...
- `network-disconnect <id: Int>`: The player with the given id has disconnected from the game, and should be removed from your player list.
- `network-assign <id: Int>`: You have been allowed into the game, this is your player id. (This is your identity disk, everything you do or learn is imprinted on this disk! :) ... )
- `network-busy`: The server already has a game going and cannot let you in at this time. (`request-join`?)
- `network-chat <id: Int> <messsage: String>`: The player with the given id, said `message`. (Added in 1.2.)
- `game-start`: A new game is starting.
- `game-restart`: This game has ended, and a new one is now starting.
- `game-play <id: Int> <vertical: Boolean> <x: Int>, <y: Int>`: The player with the given id has made a move at dot `x, y`, where `vertical` represents the line below the dot if `true` or the line to the right of the dot if `false`. My code still ran the checks to make sure the move was valid, so if you ever desynced with the server... uh oh - I never encountered an issue like that though.
- `game-stop`: This game has ended. It is implied that we are returning to the lobby to allow new players in or just wait.
- `info-warn <warning: String>`: The server is issuing you a warning!
- `info-malformed`: The server received a command from you that it could not parse. As in, the command was valid (i.e. `game-play`) but the arguments provided were bad (maybe it tried to parse part of the string as an integer and that threw an exception).

### Client to Server
- `player-rename <name: String>`: Request the client be renamed to the given name.
- `player-color <color: Int>`: Request the client color be changed to the given color.
- `network-disconnect`: Command sent as a nicety, informing the server the client is going to disconnect. Servers should probably be able to handle a client disconnecting without sending this message though for obvious reasons...
- `network-chat <message: String>`: Tell the server you are sending `message` to the chat. (Added in 1.2.)
- `game-play <vertical: Boolean> <x: Int>, <y: Int>`: Client wants to make a move at dot `x, y`, where `vertical` means the line below the dot if `true` and the line to the right of the dot if `false`.
- `request-start` and `request-restart`: Both have the same meaning, and that is to ask the server to start a new game. How the server handles this is up to them (though my approach at the time was to just send a chat message telling everyone they wanted to start - or at least once I added chat).
- `request-stop`: Client wishes to stop the game - implied that they want to return to the lobby and allow new people to join.
- `request-join`: Informs the server a client is attempting to connect (sent after receiving `network-busy`). I know. It doesn't make any sense, since the server already knows this? I guess the thinking is, they can leave after `network-busy`, but if they really want to play, they can tell the server. (No doubt I sent a chat message once that feature was added.)