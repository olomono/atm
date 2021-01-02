# Test Cases for Automatic Trust Management (ATM)

These test cases can be used for verifying the correctness of an implementation of the XMPP Extension Protocol (XEP) Automatic Trust Management (ATM).

A1, A2 and A3 are devices of the XMPP account A.
B1 and B2 are devices of the XMPP account B.
The implementation which is tested runs on device A1.

## Manual Authentication

Manual Authentication for...

### B1's key

Case | Result
---|---
B1's key is stored | Authenticate B1's key and send trust message for B1's key to own devices.
B1's key is not stored | Store authentication data for B1's key.

### A1's key

Case | Result
---|---
A2's key is stored | Authenticate A2's key and send trust message for A2's key to own devices and to devices of contacts.
A2's key is not stored | Store authentication data for A2's key.

## Automatic Authentication

Trust message to A1 from...

### B1

Case | Result
---|---
for C1's key | Discard trust message.
for A2's key | Discard trust message.

### B1 for B2's key

Case | Result
---|---
B1's key is authenticated | Authenticate B2's key and authenticate all keys of previous trust meesages of B2's key.
B1's key is not authenticated | Cache trust message data.

### A2 for A3's key

Case | Result
---|---
A2's key is authenticated | Authenticate A3's key and authenticate all keys of previous trust meesages of A3's key.
A2's key is not authenticated | Cache trust message data.

### A2 for B1's key

Case | Result
---|---
A2's key is authenticated | Authenticate B1's key and authenticate all keys of previous trust meesages of B1's key.
A2's key is not authenticated | Cache trust message data.
