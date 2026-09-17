
React Server Components is a feature introduced in React 19 that allows components to be rendered on the server rather than the client’s browser. This provides significant performance benefits, as the server can handle computationally intensive tasks whilst sending only the rendered output to the client.

The communication between the server and client in RSC relies on a protocol called **React Flight**. This protocol handles the serialisation and deserialisation of data being transmitted between the server and client. When a client needs to invoke a server-side function (called a “Server Action”), it sends a specially formatted request containing serialised data that the server deserialises and processes.

The Flight protocol uses a specific serialisation format with type markers. For example:

- `$@` denotes a chunk reference
- `$B` denotes a Blob reference
- References can include property paths using colon separation (e.g., `$1:constructor:constructor`)

This serialisation mechanism is where the vulnerability lies. The server processes these references without properly validating that the requested properties are legitimate exports from the intended module.