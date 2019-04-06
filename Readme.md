# Automatic Trust Transfer (ATT)

ATT is an XMPP Extension Protocol (XEP) which aims to simplify secure end-to-end encrypted communication using [OMEMO](https://xmpp.org/extensions/xep-0384.html) by reducing the number of manually made key authentications.

## Resources

### Specification (XEP)

#### Current Working Version

* [HTML](https://olomono.github.io/xeps/build/xep-xxxx-automatic-trust-transfer.html)
* [XML - Original](https://github.com/olomono/xeps/tree/automatic-trust-transfer)

#### To XMPP Standards Foundation (XSF) Submitted Version

* [HTML](https://xmpp.org/extensions/inbox/automatic-trust-transfer.html)
* [XML - Original](https://github.com/xsf/xeps/blob/master/inbox/automatic-trust-transfer.xml)
* [Pull Request](https://github.com/xsf/xeps/pull/763)

### Implementation

* [Algorithm (pseudocode)](https://github.com/olomono/att/blob/master/algorithm.md)
* [Implementation (for the XMPP Client *Conversations*)](https://github.com/siacs/Conversations/pull/3400)

### Standardization

* [Discussion (on the mailing list)](https://mail.jabber.org/pipermail/standards/2019-March/035945.html)

## Process

1. After a **manual authentication** like QR code scanning,
1. the trust in specific keys is transferred by so called **trust messages** to other devices with already authenticated keys.
1. Afterwards, the devices which received such an OMEMO encrypted trust message from a device with an already authenticated key can **automatically authenticate** the key given by the trust message.
