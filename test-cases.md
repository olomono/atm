# Test Cases for Automatic Trust Management (ATM)

These test cases can be used for verifying the correctness of an implementation of the XMPP Extension Protocol (XEP) Automatic Trust Management (ATM).

A1, A2 and A3 are devices of the XMPP account A.
B1 and B2 are devices of the XMPP account B.
C1 is a device of the XMPP account C.
The implementation being tested runs on device A1.

## Manual Authentication / Distrusting

Manual authentication / distrusting for...

Case | Result
---|---
B1's key | Authenticate / Distrust B1's key, send trust message for B1's key to own devices with authenticated keys and for keys of own devices to B1
A2's key | Authenticate / Distrust A2's key, send trust message for A2's key to all devices with authenticated keys and for keys of all devices to A2

## Automatic Authentication / Distrusting

Trust message to A1 from...

### B1

Case | Result
---|---
for C1's key | Discard trust message
for A2's key | Discard trust message

### B1 for B2's key

Case | Result
---|---
B1's key is authenticated | Authenticate / Distrust B2's key and authenticate / distrust all keys of previous trust messages from B2
B1's key is not authenticated | Store trust data for B2's key

### A2 for A3's key

Case | Result
---|---
A2's key is authenticated | Authenticate / Distrust A3's key and authenticate / distrust all keys of previous trust messages from A3
A2's key is not authenticated | Store trust data for A3's key

### A2 for B1's key

Case | Result
---|---
A2's key is authenticated | Authenticate / Distrust B1's key and authenticate / distrust all keys of previous trust messages from B1
A2's key is not authenticated | Store trust data for B1's key
