# Algorithm of Automatic Trust Management (ATM)

This algorithm should serve as a guidance for implementing the XMPP Extension Protocol (XEP) Automatic Trust Management (ATM).

## Definitions

Term | Definition
---|---
`AND` | logical *and* (`∧`)
`OR` | logical *or* (`∨`)
`!` | logical *not* (`¬`)
`//` | comment
`<term>` | placeholder which must be replaced by a real term
(`<termA>`, `<termB>`) | map / associative array of `<termA>` and `<termB>`
message | XMPP message
key | public long-term key
bare JID | `<user>@<server>`
XMPP URI | `xmpp:<bare-jid>?query-type;<key-1>=<value-1>;<key-2>=<value-2>;...;<key-N>=<value-n>`
trust message URI | `xmpp:<bare-jid>?trust-message;encryption=<encryption-namespace>;trust=<key-fingerprint-1>;trust=<key-fingerprint-2>;<...>;trust=<key-fingerprint-n>;...>distrust=<key-fingerprint-n+1>;distrust=<key-fingerprint-n+2>;...;distrust=<key-fingerprint-n+m>`
encryptionNamespace | namespace of the key's encryption protocol
sender | sender of the trust message (bare JID)
senderFingerprint | fingerprint of the sender's key
recipient | recipient of the trust message (bare JID)
account | the user's own account (bare JID)
contact | a user's contact (bare JID)
contacts | the user's contacts (bare JIDs)
owner | owner of keys represented by their fingerprints in trust message (bare JID, `jid` attribute in trust message element `<key-owner/>`)
fingerprintsOfAuthenticatedKeysOfContact | fingerprints of authenticated keys of the corresponding contact
fingerprintsOfAuthenticatedKeysOfOwner | fingerprints of authenticated keys of the corresponding owner
fingerprints | fingerprints contained in trust message
fingerprintsForAuthentication | fingerprints in `<trust/>` element
fingerprintsForDistrusting | fingerprints in `<distrust/>` element
fingerprintsForTrustAction | fingerprints used to authenticate or distrust their corresponding keys
storage | local storage for data (e.g., a database)
trust | trust in key (`true` for authentication, `false` for distrusting)

## Trust Levels

A key can have exactly one of the following trust levels:

1. untrusted
1. trusted
	1. manually trusted (e.g., by clicking a button) OR automatically trusted (e.g., by the client for all keys of a bare JID until one of it is authenticated)
	1. manually authenticated (e.g., by QR code scanning) OR automatcially authenticated (e.g., by ATM)

## Manual Authentication or Distrusting

### Creating a Trust Message URI

A trust message can be represented by an XMPP URI which can be shared as a QR code.
Such a QR code can be scanned by persons who trust the QR code author to authenticate or distrust the keys corresponding to the fingerprints in the trust message.

```
// URI for Authentication
if (there are authenticated keys with fingerprints of owner for encryption)
	return createTrustMessageUri(owner, fingerprints, encryption, true)
```

```
// URI for Distrusting
if (there are untrusted keys with fingerprints of owner for encryption)
	return createTrustMessageUri(owner, fingerprints, encryption, false)
```

### Reading a Trust Message URI (e.g., by QR Code Scan, NFC, Wifi, Bluetooth etc.)

Attention: The user should only use a trust message URI from a trusted source since that source is able to initiate an authentication or revocation for keys which do not belong to it.

```
if (
		input is XMPP URI
		AND XMPP URI is URI of trust message
		AND user confirmed to use that URI for authenticating or distrusting the keys of the owner specified in the URI
)

authenticate(owner, fingerprintsForAuthentication, true)
distrust(owner, fingerprintsForDistrusting, true)
```

## Automatic Authentication or Distrusting

### Receiving a Message

An XMPP message is a trust message if it has a [trust message structure](https://xmpp.org/extensions/xep-0434.html#trust-message-structure).
ATM uses [encrypted trust messages](https://xmpp.org/extensions/xep-0434.html#use-cases-encrypted-trust-message).

```
if (
	message is signed by a signing mechanism
	AND message is encrypted
	AND message has trust message structure
	AND (sender is recipient OR owner is sender)
)
	if (key of senderFingerprint is authenticated)
		authenticate(owner, fingerprints, false)
		distrust(owner, fingerprints, false)
	else
		cacheTrustMessageData(owner, fingerprint, senderFingerprint, true)
		cacheTrustMessageData(owner, fingerprint, senderFingerprint, false)
```

## Subroutines

### createTrustMessageUri(owner, fingerprints, encryptionNamespace, trust)

```
uri = "xmpp:" + owner + "?trust-message;encryption=" + encryptionNamespace
if (trust)
	key = "trust"
else
	key = "distrust"
for (fingerprint in fingerprints)
	uri = uri + ";" + key + "=" + fingerprint
return uri
```

### authenticate(owner, fingerprints, sendTrustMessage)

If only the trust level *trusted* instead of also *authenticated* should be used, it is required to check if the current authentication is the first one.
If it is the first one, the distrusting of all keys which were not authenticated will only happen at the first time.
If a user manually trusts a key afterwards, that one will be seen as authenticated.

```
for (fingerprint in fingerprints)
	if (key of fingerprint is in storage)
		if (key of fingerprint belongs to owner AND key of fingerprint is not authenticated)
			authenticate key by fingerprint
			add fingerprint to fingerprintsForTrustAction
			authenticateOrDistrustWithCachedTrustMessageData(fingerprint)
	else
		preAuthenticate(owner, fingerprint)
if (fingerprintsForTrustAction has at least one fingerprint)
	if (there is at least one automatically trusted key of owner OR this is the first authentication of owner's key)
		distrust all keys of owner which are not yet authenticated
	if (sendTrustMessage)
		sendTrustMessage(owner, fingerprintsForTrustAction, true)
```

### distrust(owner, fingerprints, sendTrustMessage)

```
for (fingerprint in fingerprints)
	if (key of fingerprint is in storage)
		if (key of fingerprint belongs to owner AND key of fingerprint is trusted)
			distrust key by fingerprint
			add fingerprint to fingerprintsForTrustAction
			deleteTrustMessageDataForSender(fingerprint)
	else
		preDistrust(owner, fingerprint)
if (fingerprintsForTrustAction is not empty AND sendTrustMessage)
		sendTrustMessage(owner, fingerprintsForTrustAction, false)
```

### authenticateOrDistrustWithCachedTrustMessageData(senderFingerprint)

```
// Authentication
authenticateOrDistrustWithCachedTrustMessageData(senderFingerprint, true)

// Distrusting
authenticateOrDistrustWithCachedTrustMessageData(senderFingerprint, false)
```

### authenticateOrDistrustWithCachedTrustMessageData(senderFingerprint, trust)

```
for (owner, fingerprints) in loadTrustMessageData(senderFingerprint, trust))
	for (fingerprint in fingerprints)
		add fingerprint to fingerprintsForTrustAction
		deleteTrustMessageDataForFingerprint(fingerprint)
	if (trust)
		authenticate(owner, fingerprintsForTrustAction, false)
	else
		distrust(owner, fingerprintsForTrustAction, false)
```

### cacheTrustMessageData(owner, fingerprint, senderFingerprint, trust)

```
if (trust message data for owner, fingerprint, senderFingerprint, trust exists)
	return
if (trust message data for owner, fingerprint, senderFingerprint, !trust exists)
	update data in storage for owner, fingerprint, senderFingerprint with !trust
if (trust message data for owner, fingerprint, !trust exists)
	delete data in storage for owner, fingerprint
insert trust message data into storage for owner, fingerprint, senderFingerprint, trust
```

### loadTrustMessageData(senderFingerprint, trust)

```
return (owners, fingerprints) from storage for matching senderFingerprint and trust
```

### deleteTrustMessageDataForFingerprint(fingerprint)

```
delete data from storage for matching fingerprint
```

### deleteTrustMessageDataForSender(senderFingerprint)

```
delete data from storage for matching senderFingerprint
```

### preAuthenticate(owner, fingerprint)

```
add owner and fingerprint to storage so that corresponding keys which are added later are automatically authenticated
```

### preDistrust(owner, fingerprint)

```
add owner and fingerprint to storage so that corresponding keys which are added later are automatically distrusted
```

### sendTrustMessage(owner, fingerprints, trust)

```
if (owner is account)
	deliveredViaMessageCarbons = false
	for (contact in contacts)
		if (contact has at least one authenticated key)
			sendTrustMessage(account, fingerprints, trust, contact)
			deliveredViaMessageCarbons = true
		if (account has at least one authenticated key AND trust)
			sendTrustMessage(contact, fingerprintsOfAuthenticatedKeysOfContact, true, account)
	if (!deliveredViaMessageCarbons AND fingerprints are less than authenticated keys of account)
		sendTrustMessage(account, fingerprints, trust, account)
else
	if (account has at least one authenticated key)
		sendTrustMessage(owner, fingerprints, true, account)
	if (owner has at least one authenticated key AND trust)
		sendTrustMessage(account, fingerprintsOfAuthenticatedKeysOfOwner, true, owner)
```

### sendTrustMessage(owner, fingerprints, trust, recipient)

```
send a message with createTrustMessage(owner, fingerprints, trust) as its body to recipient and copies (via Message Carbons) for own devices only if they have authenticated keys
```


