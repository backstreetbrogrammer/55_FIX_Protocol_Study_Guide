# FIX Protocol Study Guide

> This is the study guide for Financial Information eXchange (FIX) protocol.

## Table of contents

1. [What is FIX protocol?]()
2. [Technical Specifications]()
3. [FIX Session Layer]()
4. [FIX Application Layer]()

---

## 1. What is FIX protocol?

The **Financial Information eXchange (FIX)** protocol is a messaging standard developed specifically for the real-time
electronic exchange of securities' transactions.

The FIX protocol specification was originally authored in 1992 by Robert "Bob" Lamoureux and Chris Morstatt to enable
electronic communication of equity trading data between Fidelity Investments and Salomon Brothers.

At the time, this information was communicated verbally over the **_telephone_**.

Fidelity realized that information from their broker-dealers could be routed to the wrong trader, or simply lost when
the parties hung up their phones.

It wanted such communications to be replaced with **_machine-readable_** data which could then be shared among traders,
analyzed, acted on and stored.

![FIXSampleFlow](FIXSampleFlow.PNG)

Reference: [FIX Trading community](https://www.fixtrading.org/)

---

## 2. Technical Specifications

Originally, the FIX standard was monolithic, including application layer semantics, message encoding, and session layer
in a single technical specification.

In other words, FIX was an all-in-one protocol standard that defines:

- Message **Encoding**
- **Session** Layer Messaging Protocol
- **Application** Layer Messaging Protocol
- Domain Model (aka **Data Dictionary**) for Orders, Executions, Market Data, Allocations, etc.

The most popular FIX versions `4.2` and `4.4` use this monolithic technical specification until FIX `5.0`, which
separates the FIX **_session_** layer from the FIX **_application_** layer.

### Message Encoding

Message encoding, called **Presentation Layer** in the **Open Systems Interconnection model** (`OSI` model), is
responsible for the wire format of messages.

In the seven-layer OSI model of computer networking, the presentation layer is **layer 6** and serves as the data
translator for the network. It is sometimes called the **syntax layer**.

![OSI_model](OSI_model.PNG)

**_Tagvalue encoding_**

The original FIX message encoding is known as `tagvalue` encoding.

Each field consists of a unique numeric tag and a value.

The tag identifies the field semantically. Therefore, messages are self-describing.

`Tagvalue` encoding is character-based, using `ASCII` codes.

**_FIX tagvalue message format_**

A message is composed of a **header**, a **body**, and a **trailer**.

The message fields are separated by the [start of heading](https://en.wikipedia.org/wiki/C0_and_C1_control_codes#SOH)
(SOH) character (ASCII `0x01`).

Up to `FIX.4.4`, the header contains **three** fields: `8 (BeginString)`, `9 (BodyLength)`, and `35 (MsgType)`.

From `FIXT.1.1` / `FIX.5.0`, the header contains five or six fields: `8 (BeginString)`, `9 (BodyLength)`,
`35 (MsgType)`, `49 (SenderCompID)`, `56 (TargetCompID)` and the optional `1128 (ApplVerID)`.

The content of the **message body** is defined by the message type (`35 MsgType`) in the **header**.

The **trailer** contains the **last** field of the message, `10 (Checksum)`, always expressed as a three-digit number
(e.g. `10=002`).

Example of a FIX message: **Execution Report** (Pipe character is used to represent `SOH` character: `\u0001`)

```
8=FIX.4.2 | 9=178 | 35=8 | 49=PHLX | 56=PERS | 52=20071123-05:30:00.000 | 11=ATOMNOCCC9990900 | 20=3 | 150=E | 39=E | 55=MSFT | 167=CS | 54=1 | 38=15 | 40=2 | 44=15 | 58=PHLX EQUITY TESTING | 59=0 | 47=C | 32=0 | 31=0 | 151=15 | 14=0 | 6=0 | 10=128 | 
```

**_Body_**

FIX messages are formed from several fields; each field has a **tag-value pairing** that is separated from the next
field by a delimiter SOH (`0x01`).

The **tag** is an **integer** that indicates the meaning of the field.

The value is an **array of bytes** that hold a specific meaning for the particular tag.

For example, tag `48` is `SecurityID`, a string that identifies the security.

Tag `22` is `IDSource`, an integer that indicates the identifier class being used.

The values may be in plain text or encoded as pure binary (in which case the value is preceded by a length field).

The FIX protocol defines meanings for most tags, but leaves a range of tags reserved for private use between consenting
parties => these tags are also called **custom** tags.

The FIX protocol also defines sets of fields that make a particular message; within the set of fields, some will be
**mandatory** and others **optional**.

The ordering of fields within the message is generally unimportant, however, **repeating groups** are preceded by a
count, and encrypted fields are preceded by their length.

The message is broken into three distinct sections: the **head**, **body** and **tail**.

Fields must remain within the correct section and within each section, the position may be important as fields can act
as delimiters that stop one message from running into the next.

The final field in any FIX message is `tag 10 (checksum)`.

There are two main groups of messages: **admin** and **application**.

The **admin** messages handle the basics of a **FIX session**.

They allow for a **session** to be started and terminated and for recovery of missed messages.

The **application** messages deal with the sending and receiving of trade-related information such as an order request
or information on the current state and subsequent execution of that order.

**_Body length_**

The body length is the character count starting at `tag 35` (included) all the way to `tag 10` (excluded), including
trailing SOH delimiters.

The example below (displayed with SOH delimiters as `'|'`) has a body length of `65`:

![FIXBodyLength](FIXBodyLength.PNG)

**_Checksum_**

The checksum of a FIX message is always the **last** field in the message, with `tag 10` and a `3` character value.

It is given by **summing** the **ASCII** value of all characters in the message (except for the checksum field itself),
then modulo `256`.

For example, in the message above, the summation of all ASCII values (including the SOH characters with ASCII value `1`)
results in `4158`.

Performing the modulo operation gives the value `62`.

Since the checksum is composed of three characters, this results in `10=062`.

### Sample FIX log

Sample FIX log containing various FIX messages:

[fix_log1.txt](https://github.com/backstreetbrogrammer/13_TopCodingInterviewSolutionsInJava/blob/main/src/main/resources/fix_log1.txt)

One way to remember or refer all these FIX tags is to use these two sites:

- [B2BITS](https://btobits.com/fixopaedia/index.html) - this is my personal favorite
- [FIXimate](https://fiximate.fixtrading.org/)

