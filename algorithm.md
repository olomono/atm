# Algorithm of Automatic Trust Transfer (ATT)

This algorithm should serve as a guidance for implementing the XMPP Extension Protocol (XEP) Automatic Trust Transfer (ATT).

## Definitions

Term | Definition
---|---
`AND` | logical "and" (*∧*)
`OR` | logical "or" (*∨*)
`!` | logical "not" (*¬*)
`//` | comment
`<term>` | placeholder which must be replaced by a real term
(<termA>, <termB>) | map / associative array of <termA> and <termB>
message | XMPP message
key | OMEMO public identity key
bare JID | `<user>@<server>`
XMPP URI | `xmpp:<bare-jid>?action;<action-key-1>=<value-1>;<action-key-2>=<value-2>;...;<action-key-N>=<value-n>`
trust message URI | `xmpp:<bare-jid>?omemo-trust;auth=<omemo-key-fingerprint-1>;auth=<omemo-key-fingerprint-2>;<...>;auth=<omemo-key-fingerprint-n>;...>revoke=<omemo-key-fingerprint-n+1>;revoke=<omemo-key-fingerprint-n+2>;...;revoke=<omemo-key-fingerprint-n+m>`
sender | sender of the trust message (bare JID)
senderFingerprint | fingerprint of the sender's key
recipient | recipient of the trust message (bare JID)
account | the user's own account (bare JID)
contact | a user's contact (bare JIDs)
contacts | the user's contacts (bare JIDs)
fingerprintsOfAuthenticatedKeysOfContact | fingerprints of authenticated keys of the corresponding contact
owner | owner of keys contained in trust message (bare JID, `<bare-jid>` in trust message URI)
fingerprintsOfAuthenticatedKeysOfOwner | fingerprints of authenticated keys of the corresponding owner
fingerprints | fingerprints contained in trust message
fingerprintsForAuthentication | fingerprints after `auth=` action keys in a trust message URI
fingerprintsForRevocation | fingerprints after `revoke=` action keys in a trust message URI
fingerprintsForTrustAction | fingerprints which should be used to authenticate their corresponding keys or to revoke the trust in their corresponding keys
storage | local storage for data (e.g., a database)
trust | trust in key (`true` for authentication, `false` for revocation)

## Trust Levels

A key can have exactly one of the following trust levels:

1. untrusted
1. trusted
	1. automatically (blindly) trusted (witout user interaction) OR manually trusted (by user interaction)
	1. authenticated

## Manual Authentication or Revocation

### Reading a Trust Message URI (e.g., by QR Code Scan, NFC, Wifi, Bluetooth etc.)

* Attention: The user should only use a trust message URI from a trusted source since that source is able to initiate an authentication or revocation for keys which do not belong to it.

```
if (
		input is XMPP URI
		AND XMPP URI is URI of trust message
		AND user confirmed to use that URI for authentication or revocation relating to the keys of the owner specified in the URI
)

authenticate(owner, fingerprintsForAuthentication, true)
revoke(owner, fingerprintsForRevocation, true)
```

## Automatic Authentication or Revocation

### Receiving a Message

```
if (
		message is OMEMO encrypted
		AND message body is XMPP URI
		AND XMPP URI is trust message URI
		AND (sender is recipient OR owner is sender)
)
	if (key of senderFingerprint is authenticated)
		authenticate(owner, fingerprints, false)
		revoke(owner, fingerprints, false)
	else
		cacheTrustMessageData(owner, fingerprint, senderFingerprint, true)
		cacheTrustMessageData(owner, fingerprint, senderFingerprint, false)
```

### authenticate(owner, fingerprints, sendTrustMessage)

* If only the trust level *trusted* instead of also *authenticated* should be used, it is required to check if the current authentication is the first one.
If it is the first one, the distrusting of all keys which were not authenticated will only happen at the first time.
If a user manually trusts a key afterwards, that one will be seen as authenticated.

```
distrustAllUnauthenticatedKeys = true if owner has at least one blindly trusted key OR this is the first authentication of an owner's key
for (fingerprint in fingerprints)
	if (key of fingerprint is in storage)
		if (key of fingerprint belongs to owner AND key of fingerprint is not authenticated)
			authenticate key by fingerprint
			add fingerprint to fingerprintsForTrustAction
			authenticateOrRevokeWithCachedTrustMessageData(fingerprint)
	else
		preAuthenticate(owner, fingerprint)
if (fingerprintsForTrustAction has at least one fingerprint)
	if (distrustAllUnauthenticatedKeys)
		distrust all keys of owner which are not yet authenticated
	if (sendTrustMessage)
		sendTrustMessage(owner, fingerprintsForTrustAction, true)
```

### revoke(owner, fingerprints, sendTrustMessage)

* Is there a use case when it is important to know if the trust in a key is revoked or is it sufficient to know that a key is untrusted?
If the latter is true, it is sufficient to use the term *untrusted* and not also *revoked*.

```
for (fingerprint in fingerprints)
	if (key of fingerprint is in storage)
		if (key of fingerprint belongs to owner AND key of fingerprint is trusted)
			distrust key by fingerprint
			add fingerprint to fingerprintsForTrustAction
			deleteTrustMessageDataForSender(fingerprint)
	else
		preRevoke(owner, fingerprint)
if (fingerprintsForTrustAction is not empty AND sendTrustMessage)
		sendTrustMessage(owner, fingerprintsForTrustAction, false)
```

### authenticateOrRevokeWithCachedTrustMessageData(senderFingerprint)

```
// Authentication
authenticateOrRevokeWithCachedTrustMessageData(senderFingerprint, true)

// Revocation
authenticateOrRevokeWithCachedTrustMessageData(senderFingerprint, false)
```

### authenticateOrRevokeWithCachedTrustMessageData(senderFingerprint, trust)

```
for (owner, fingerprints) in loadTrustMessageData(senderFingerprint, trust))
	for (fingerprint in fingerprints)
		add fingerprint to fingerprintsForTrustAction
		deleteTrustMessageDataForFingerprint(fingerprint)
	if (trust)
		authenticate(owner, fingerprintsForTrustAction, false)
	else
		revoke(owner, fingerprintsForTrustAction, false)
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

### preRevoke(owner, fingerprint)

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
send a message with createTrustMessageBody(keysOwner, fingerprints, trust) as its body to recipient and copies (via Message Carbons) for own devices only if they have authenticated keys
```

### createTrustMessageBody(owner, fingerprints, trust)

```
trustMessageBody = "xmpp:" + owner + "?omemo-trust"
if (trust)
		actionKey = "auth"
	else
		actionKey = "revoke"
for (fingerprint in fingerprints)
	trustMessageBody = trustMessageBody + ";" + actionKey + "=" + fingerprint;
}
return trustMessageBody
```
