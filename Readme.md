# Automatic Trust Management (ATM)

ATM is an XMPP Extension Protocol (XEP) that aims to simplify secure end-to-end encrypted communication by automating the authentication and distrusting of public long-term keys.

## Resources

### Specification (XEP)

#### Current Working Version

* [HTML](https://olomono.github.io/xeps/preview/xep-0450.html)
* [XML - Original](https://github.com/olomono/xeps/blob/atm/xep-0450.xml)

#### To XMPP Standards Foundation (XSF) Submitted Version

* [HTML](https://xmpp.org/extensions/xep-0450.html)
* [XML - Original](https://github.com/xsf/xeps/blob/master/xep-0450.xml)

### Implementation

* [Algorithm (Pseudocode)](algorithm.md)
* [Test Cases](test-cases.md)
* [QXmpp](https://github.com/qxmpp-project/qxmpp)
* [Kaidan](https://invent.kde.org/network/kaidan)

## Process

1. After an **initial authentication** (e.g., scanning a QR code)
1. the trust in keys is communicated by [trust messages](https://xmpp.org/extensions/xep-0434.html) to other devices with already authenticated keys.
1. Afterwards, the devices that received such a trust message from a device with an already authenticated key can **automatically authenticate or distrust** the key specified by the trust message.

## Example

Here is a detailed example to demonstrate the functionality of Trust Messages (TM) and Automatic Trust Management (ATM).
To keep it simple, implementation details are omitted.
The authentications are counted in brackets (e.g., **(3)** for the third authentication).

### Baseline

Alice would like to chat end-to-end encrypted with Bob.
She uses a notebook, a tablet and a smartphone for chatting via XMPP.
Bob uses a notebook and a smartphone.

In the case of [OMEMO](https://xmpp.org/extensions/xep-0384.html), each device has an own key.
Therefore, Alice has three keys and Bob has two keys.
Alice has to authenticate Bob's keys and Bob has to authenticate Alice's keys.
Furthermore, they have to authenticate their own keys.

### Without ATM

Currently, that involves the following actions:

1. **Alice**'s notebook has to authenticate the key of her tablet **(1)**, the key of her smartphone **(2)**, the key of Bob's notebook **(3)** and the key of his smartphone **(4)**.
1. Alice's tablet has to authenticate the key of her notebook **(5)**, the key of her smartphone **(6)**, the key of Bob's notebook **(7)** and the key of his smartphone **(8)**.
1. Alice's smartphone has to authenticate the key of her notebook **(9)**, the key of her tablet (10), the key of Bob's notebook **(11)** and the key of his smartphone **(12)**.
1. **Bob**'s notebook has to authenticate the key of his smartphone **(13)**, the key of Alice's notebook **(14)**, the key of her tablet **(15)** and the key of her smartphone **(16)**.
1. Bob's smartphone has to authenticate the key of his notebook **(17)**, the key of Alice's notebook **(18)**, the key of her tablet **(19)** and the key of her smartphone **(20)**.

The whole process involves **20 authentications**.
Doing those authentications manually (e.g., by scanning QR codes containing the fingerprint of each device) is a huge task for users.

### With ATM

When you use ATM, the number of manual authentications can be reduced to a minimum while the other authentications are done automatically and in a secure manner.
The authentications that ATM needs to automate the remaining authentications are the *initial authentications*.
In the following, only the initial authentications are counted.

An example procedure making use of QR code scanning could be the following:

1. **Alice** scans with her notebook the QR code of her smartphone to authenticate her smartphone's key **(1)**.
1. Alice scans with her tablet the QR code of her smartphone to authenticate her smartphone key **(2)**.
1. Alice scans with her smartphone the QR code of her notebook to authenticate her notebook's key **(3)**.
1. Alice scans with her smartphone the QR code of her tablet to authenticate her tablet's key **(4)**.
	1. Her smartphone automatically sends a trust message for the key of her notebook to her tablet.
	1. Her tablet uses the trust message to automatically authenticate the key of her notebook.
	1. Her smartphone automatically sends a trust message for the key of her tablet to her notebook.
	1. Her notebook uses the trust message to automatically authenticate the key of her tablet.
1. **Bob** scans with his notebook the QR code of his smartphone to authenticate his smartphone's key **(5)**.
1. Bob scans with his smartphone the QR code of his notebook to authenticate his notebook's key **(6)**.
1. Bob scans with his smartphone the QR code of Alice's smartphone to authenticate her smartphone's key **(7)**.
	1. His smartphone automatically sends a trust message for the key of Alice's smartphone to his notebook.
	1. His notebook uses the trust message to automatically authenticate the key of Alice's smartphone.
	1. His smartphone automatically sends a trust message for the key of his notebook to Alice's smartphone.
	1. **Alice**'s smartphone uses the trust message to automatically authenticate the key of Bob's notebook.
1. Alice scans with her smartphone the QR code of Bob's smartphone to authenticate his smartphone's key **(8)**.
	1. Her smartphone automatically sends a trust message for the key of Bob's smartphone to her notebook and tablet.
	1. Her notebook and tablet use the trust message to automatically authenticate the key of Bob's smartphone.
	1. Her smartphone automatically sends a trust message for the key of her notebook and the key of her tablet to Bob's notebook and smartphone.
	1. **Bob**'s notebook and smartphone use the trust message to automatically authenticate the key of Alice's notebook and the key of her tablet.

### Result

**8 authentications** cannot be automated in a secure manner.
They are the *initial authentications*.
The remaining **12 authentications** can be done automatically without reducing the security.
The security could be even improved as said in my previous message in comparison to manual procedures like looking at two fingerprints and comparing them.

### Not only for OMEMO

The example describes the case where each device has an own key.
The case where each chat partner has only one key used by all devices (e.g., with [OpenPGP for XMPP](https://xmpp.org/extensions/xep-0373.html)) represents a subset.
ATM can handle that as well by reducing the number of manual authentications for multiple contacts.

The following example should explain how:

1. Alice has three contacts.
1. She has already authenticated their keys with her smartphone and would like to use a new notebook.
1. She can simply scan with her smartphone the QR code of her notebook.
1. Her smartphone automatically sends a trust message for the key of each contact to her new device.
1. As soon as Alice scanned with her notebook the QR code of her smartphone, her notebook uses the trust message to automatically authenticate the keys of all three contacts.
