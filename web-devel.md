A web app is broken to frontend *(obvious)* and backend *(at the least because there's always code you don't want to be seen by browsers)*. The two communicate via http protocol *(whether encrypted or not)*.

# http/https misc

* raw protocol methods are plain-text with lines ending with `\r\n`, where:
  1. 1-st line: space-separated method name, URL, protocol version. Example: `GET /image/logo.png HTTP/1.1`
  2. one or more header lines *(`Host` is mandatory, the rest are not)*, e.g.:
     ```
     Host: www.example.com
     Accept-Language: en
     ```
  3. empty line
  4. optionally: message body.
* protocol is stateless, so a user session is managed with cookies.
* methods used most frequently:
  * `GET`: should only retrieve data and have no other effect.
  * `HEAD`: like `GET` but for metadata. E.g. retrieving data length.
  * `POST`: asking server to process some data and mutate some state.
  * `PUT`: like `POST` but allows specifying target location and is uncacheable.
* methods result in reply from the server containing a code, and optionally some other fields and HTML or other data depending on the circumstances.

# javascript misc

JS has no static typing, so you don't want to develop in it directly. Instead it is like assembly of the web *(not to be confused with WebAssembly)*, and you'd code in another language that will get translated to JS. Of others: there's TypeScript, but its typing system is just terrible *(broken immutability, equality tests not failing for distinct types…)*. Next one is PureScript. It has excellent type system but might be complicated for newbies. So basically, web-devel here is like the tale of two stools.

PureScript [has a separate file](purescript.md).
