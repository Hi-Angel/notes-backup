# pidgin plugin

There's a good hello-world https://bitbucket.org/pidgin/skeleton-libpurple/src It's a "core" plugin, if you wanna a protocol plugin, replace `PURPLE_PLUGIN_STANDARD` with `PURPLE_PLUGIN_PROTOCOL`, and watch for errors in debug window of pidgin.

Good example of a protocol plugin is https://github.com/matrix-org/purple-matrix/

In the simplest case for a protocol-plugin Pidgin automagically creates a window with settings like account, password, etc.

libpurple docs are available online thanks to doxygen. TCP connection is at network.h. TLS is at sslcon.h. An example of http connection can be found at `jabber_bosh_connection_new()` of pidgin source code *(it's from a jabber)*.

## possible rakes

1. Pidgin reloads plugins every time you load a window with plugin list, however there's a bug in that it doesn't reload ones that changed. So you need to either of 1. fix that pidgin bug, 2. restart pidgin everytime, 3. make every time a new name for the plugin *(if you're writing a protocol plugin, edit also `PurplePluginInfo::id`)*.
2. For using C++ don't forget to `export "C" {}` around callbacks.

`g_log: purple_presence_set_status_active: assertion 'status != NULL' failed` — for me it was when I didn't implement `PurplePluginProtocolInfo::status_types`. It's called even before the login callback.

# Writing Trillian plugin
## general

Possible troubles:
1. Dunno know how to extract the protocol
2. The protocol is encrypted
3. The protocol is obfuscated

So, what do I need before bidding:
1. Write a simple plugin implementing the most trivial *(but necessarily network-related)* functionality of Trillian. Perhaps sending a message to yourself if it's allowed, otherwise to someone else. Or setting an avatar, then checking in native trillian client.

## Documentation

Trillian has docs at https://trillian.im/impp/ so it's possible their protocol is not that closed. Some info from there:

* TLV-based
* Authorization-based contact lists: only users you approve can track your presence.
* Capability-based online and offline messaging: devices can be shielded from unsupported message types *(probably due to TLV)*
* Clients are required to use TLS and may optionally utilize DEFLATE-based compression.
* Extendable: Trillian-specific extensions for continuous client, cloud history, and more.
* Leverage of existing standards when applicable: TLS, SIP, ICE, TURN, RTP, etc.
* Invisibility with allow and block lists.
* Intelligent presence: idle and mobile devices are tracked to determine the best status to advertise.
* Multi-device awareness: state and messages are shared between all connected devices.
* Server-backed, persistent group chats.
* User avatars.

Communication is through `tlv_packet`s:

```
#pragma pack(push, 1)

// Array of distinct tlv_unitN is the content of the block in tlv_value_header. A union doesn't work
// here because it's always aligned to 4 bytes. The active member depends on the type
struct tlv_unit16 {
    uint16_t type;
    uint16_t value_sz;
    uint8_t  value[];
};

struct tlv_unit32 {
    uint16_t type;
    uint32_t value_sz;
    uint8_t  value[];
};

struct tlv_value_header {

    /* 1. A request. If the response, indication, and error bits are all set to 0, the
       message is a request. Requests are the only type of messages sent by clients to
       servers.

       2. A response. Responses are sent from server to client and are always tied to
       a particular request. The sequence number of a response will correspond to the
       request the response belongs to.

       3. An indication. Indications are "server-initiated" messages not tied to any
       particular client request. The sequence number of an indication will always be
       set to 0.

       4. An error. Errors are typically tied to a particular request but MAY be
       stateless. The sequence number will either be 0 or the sequence number of the
       request that resulted in an error.*/
    uint16_t flags;

    /* The most significant bit of a message family value is reserved. The allowable
       range of values for families is therefore 0-32767. Within that range:

       1. The values from 0-16383 are reserved for the core IMPP protocol.

       2. The values from 16384-32767 are reserved for extensions and are not defined
       as a part of the core IMPP protocol. Clients and servers MUST mark all messages
       from extended families with the extension bit. */
    uint16_t family;

    /* The most significant bit of a message type value is reserved. The allowable
       range of values for types is therefore 0-32767. Within that range:

       1. The values from 0-16383 are reserved for the core IMPP protocol.

       2. The values from 16384-32767 are reserved for extensions and are not defined
       as a part of the core IMPP protocol. Clients and servers MUST mark all messages
       from extended types with the extension bit. */
    uint16_t msg_type;

    /* TLV messages are sequenced and MUST be sent in sequenced order. Messages
       received by the server are processed in per-family FIFO order. The sequence
       value itself starts at a random value and is incremented by one for every
       message regardless of family. For example, a client may send three messages:

       1. LISTS::CONTACT_ADD with sequence 100.
       2. LISTS::CONTACT_REMOVE with sequence 101.
       3. PRESENCE::SET with sequence 102.

       In this example, the server will process the first request, hold the second
       request until it's finished with the first, and process the third request
       immediately. Once the server responds to request 100, (which may involve
       backend communication with a database, thereby requiring a wait period) it is
       then allowed to continue processing messages within the LISTS family. Responses
       from the server can therefore come out-of-order. Clients MUST store the
       sequence associated with a message and be prepard to act on its response at any
       time.*/
    uint32_t sequence;
    uint32_t block_sz;
    uint8_t  block[];
};

struct tlv_packet{
    unsigned char magic; // should always be 0x6f
    enum channel : uint8_t {
        version = 0x1,
        tlv     = 0x2
    };
    union {
        tlv_value_header msg;
        uint16_t protocol_version;
    }u;
};
#pragma pack(pop)
```

`Device` in their terminology is a client, whether mobile or PC.

There're request examples in a bytecode, and the appendix has some of the codes.

When a client sends a request and that request fails, the server MUST respond with an error. All errors sent by the server MUST include the special errorcode TLV with type 0x0000 and a 16-bit errorcode value.

## compression

Compression is using deflate method *(e.g. zlib)*. Compression applies only to the block of a packet.

## Analysis of sniffed Trillian session after TLS established

"payload" here abstractly refers to either whole block or "val" fields of block units. Server replies typically have `flags=response` on success, or `error` on error.

1. Client sends authenticate request with name'n'password
2. Server responds, along with some unknown payload. However I don't see the type of
   the payload nor parts of the bytecode anywhere down the dump, probably it can be
   freely ignored.
3. Client sends ping *(no payload)*
4. Server responds to ping with TIMESTAMP payload.
5. Server sends BIND response with payload: `type=DEVICENAME`, and human-readable
   `val`: my hostname with some numbers appended. There's no according client
   request, it might have happened before TLS since I didn't look that part.
6. Client sends request with `flags = extension`, with payload of 3 units; from ascii
   dump can be seen words `trillian:identities`, `trillian:astra:mynickname`,
   `trillian:contactlist`.
7. Server responds with `flags=9` *(i.e. not the response)* though with same
   sequence. Three units of `tlv::type` same as in request, and similar payload
   looking like either `[uint16,uint16,uint16]`, or `[uint32,uint16]`.
8.  Client asks `lists`,           no payload.
9.  Server responds `lists`,       no payload.
10. Client asks `group_chats`,     no payload.
11. Server responds `group_chats`, no payload.
12. Client asks `im`,              no payload.
13. Server responds `im`,          no payload.
14. Client asks `presence`,        no payload.
15. Server responds `presence`,    no payload.

After some time comes ping, same way as it did earlier. Isn't tcp enough?

# misc

Some "chat" functions `purple_serv_got_chat_in` and `purple_conversation_write_system_message`, `purple_chat_conversation_set_topic`.
