# Automatic Trust Management (ATM)

ATM is an XMPP Extension Protocol (XEP) which aims to simplify secure end-to-end encrypted communication by automating the authentication and distrusting of public long-term keys.

## Resources

### Specification (XEP)

#### Current Working Version

* [HTML](https://olomono.github.io/xeps/build/xep-xxxx-automatic-trust-management.html)
* [XML - Original](https://github.com/olomono/xeps/tree/automatic-trust-management)

#### To XMPP Standards Foundation (XSF) Submitted Version

* [HTML](https://xmpp.org/extensions/xep-0450.html)
* [XML - Original](https://github.com/xsf/xeps/blob/master/xep-0450.xml)

### Implementation

* [Algorithm (Pseudocode)](algorithm.md)
* [Test Cases](test-cases.md)
* An implementation for the XMPP Client [Kaidan](https://invent.kde.org/network/kaidan) will follow

## Process

1. After an **initial authentication** (e.g., scanning a QR code)
1. the trust in keys is communicated by [trust messages](https://xmpp.org/extensions/xep-0434.html) to other devices with already authenticated keys.
1. Afterwards, the devices which received such a trust message from a device with an already authenticated key can **automatically authenticate or distrust** the key specified by the trust message.
