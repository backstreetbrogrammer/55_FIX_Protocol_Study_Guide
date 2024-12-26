# FIX Protocol Study Guide

> This is the study guide for Financial Information eXchange (FIX) protocol.

## Table of contents

1. [What is FIX protocol?](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide?tab=readme-ov-file#1-what-is-fix-protocol)
2. [Technical Specifications](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide?tab=readme-ov-file#2-technical-specifications)
    - [Message Encoding](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide?tab=readme-ov-file#message-encoding)
    - [Sample FIX log](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide?tab=readme-ov-file#sample-fix-log)
3. [FIX Session Layer](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide?tab=readme-ov-file#3-fix-session-layer)
    - [Message Recovery](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide?tab=readme-ov-file#message-recovery)
4. [FIX Application Layer](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide?tab=readme-ov-file#4-fix-application-layer)

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

[fix_log1.txt](https://github.com/backstreetbrogrammer/55_FIX_Protocol_Study_Guide/blob/main/fix_log1.txt)

One way to remember or refer all these FIX tags is to use these two sites:

- [B2BITS](https://btobits.com/fixopaedia/index.html) - this is my personal favorite
- [FIXimate](https://fiximate.fixtrading.org/)

---

## 3. FIX Session Layer

![FixSession00](FixSession00.PNG)

![FixSession01](FixSession01.PNG)

![FixSession02](FixSession02.PNG)

FIX is a **point-to-point**, **sequenced**, **session-oriented** protocol.

This means that a FIX session must be established before any sequenced messages may be exchanged.

This is analogous to two people starting the conversation on a telephone or mobile.

![FIXConnection](FIXConnection.PNG)

FIX is meant to be agnostic to the underlying transport, although, in practice, it is almost always implemented over a
long-running **TCP** connection, where the lifecycle of the FIX session coincides with the lifecycle of the TCP
connection.

![FixSession04](FixSession04.PNG)

Although it seems that the use of FIX sequence numbers over TCP is seemingly redundant, given that TCP itself is a
sequenced, session-oriented protocol, since FIX’s sequence numbers are meant to survive **TCP disconnects**.

![FixSession05](FixSession05.PNG)

Each FIX session starts with a **Logon** message (`35=A`) and usually ends with a **Logout** message (`35=5`).

There are **two** parties to each FIX session => one is designated as the **initiator**, and the other as the
**acceptor**.

![FixSession03](FixSession03.PNG)

**_Logon (35=A)_**

![FixSession06](FixSession06.PNG)

![FixSession07](FixSession07.PNG)

Once the TCP connection is established, the **initiator** sends a **Logon** message, and the **acceptor** responds back
with its own **Logon** message.

If both of these messages are accepted, the **FIX session is established**.

**_Heartbeat (35=0)_**

Once a FIX session is established and message recovery has been completed as necessary, the parties send periodic
**Heartbeat** (`35=0`) messages to indicate to each other that they're still connected.

![FixSession15](FixSession15.PNG)

![FixSession16](FixSession16.PNG)

**_Test Request (35=1)_**

If one of the parties hasn't received a **Heartbeat** for a certain amount of time, it can issue a **Test Request**
(`35=1`) to see if the counterparty is still connected and will respond to this probe.

![FixSession17](FixSession17.PNG)

![FixSession18](FixSession18.PNG)

**_Logout (35=5)_**

![FixSession26](FixSession26.PNG)

If the counterparty does not respond in a timely manner, the session will likely be terminated (either by using a
**Logout** message or just disconnecting at the TCP level).

### Message Recovery

FIX also provides mechanisms for **recovering** lost messages through the use of sequence numbers.

Each party keeps track of its own **outgoing** sequence number, but also an expected **incoming** sequence number from
the counterparty.

![FixSession08](FixSession08.PNG)

![FixSession11](FixSession11.PNG)

![FixSession12](FixSession12.PNG)

![FixSession13](FixSession13.PNG)

![FixSession14](FixSession14.PNG)

**_Resend Request (35=2)_**

![FixSession09](FixSession09.PNG)

![FixSession10](FixSession10.PNG)

If a party sends a **Logon** with an unexpectedly high sequence number, the counterparty will issue a
**Resend Request** (`35=2`) message with the sequence numbers it is missing.

**_Sequence Reset (35=4)_**

![FixSession22](FixSession22.PNG)

![FixSession23](FixSession23.PNG)

![FixSession24](FixSession24.PNG)

![FixSession25](FixSession25.PNG)

In response to this, the first party should respond back with the missing messages, or else, it can send a
**Sequence Reset** (`35=4`) indicating that there are no "interesting" messages to resend (this could happen if all the
missed messages are session-level messages from a previous session that are irrelevant in this session).

**_Possible Resend_**

![FixSession27](FixSession27.PNG)

![FixSession28](FixSession28.PNG)

**_Rejects_**

Lastly, if a party encounters a malformed or an otherwise unacceptable message from the counterparty, it can use a
**Session-level Reject** (`35=3`) message to indicate the same.

![FixSession19](FixSession19.PNG)

![FixSession20](FixSession20.PNG)

![FixSession21](FixSession21.PNG)

**_24 x 7 Trading_**

![FixSession29](FixSession29.PNG)

![FixSession30](FixSession30.PNG)

**_A Typical FIX Session_**

![FixSession31](FixSession31.PNG)

**_Reference_**

[FIX 4.2 Session](https://btobits.com/fixopaedia/fixdic42/index.html)

![FIXSessionSummary](FIXSessionSummary.PNG)

---

## 4. FIX Application Layer

Once a FIX session is established and the sequence of negotiations has been completed, application-layer messages may be
sent.

Let’s take a look at a typical lifecycle of an **order** (in `FIX 4.2`):

- An order is typically placed using
  a [New Order Single](https://btobits.com/fixopaedia/fixdic42/message_New_Order_Single_D.html) message (`35=D`)
- The order is **acknowledged** or **rejected** using an
  [ExecutionReport](https://btobits.com/fixopaedia/fixdic42/message_Execution_Report_8.html) message (`35=8`) with
  `OrdStatus=New` (`39=0`) or `OrdStatus=Rejected` (`39=8`)
  => [OrdStatus](https://btobits.com/fixopaedia/fixdic42/tag_39_OrdStatus.html)
- Assuming the order is accepted, it may receive a **partial fill**, which is again represented using an
  `ExecutionReport` with `OrdStatus=Partially filled` (`39=1`). If the order is **fully filled**, the message will
  contain `OrdStatus=Filled` (`39=2`).
- A request to **amend** the order parameters is sent as an
  [Order Cancel/Replace Request](https://btobits.com/fixopaedia/fixdic42/message_Order_Cancel_Replace_Request_G.html)
  message (`35=G`).
- An amendment request is **accepted**, again using an `ExecutionReport` message, by specifying `ExecType=Replaced`
  (`150=5`).
  => [ExecType](https://btobits.com/fixopaedia/fixdic42/tag_150_ExecType.html)
- A request to **cancel** the order is sent as an
  [Order Cancel Request](https://btobits.com/fixopaedia/fixdic42/message_Order_Cancel_Request_F.html) message (`35=F`).
- Acceptance of the order cancellation is sent using, again, an `ExecutionReport` message with `OrdStatus=Canceled`
  (`39=4`).
- An amendment or a cancellation request can be **rejected** using an
  [Order Cancel Reject](https://btobits.com/fixopaedia/fixdic42/message_Order_Cancel_Reject_9.html) message (`35=9`).

![FIXOrderFlow](FIXOrderFlow.PNG)

### Order State Change Matrices

Each `ExecutionReport` (`39=8`) contains two fields which are used to communicate both the current state of the order as
understood by the broker (`OrdStatus`) and the purpose of the message (`ExecType`).

In an execution report the `OrdStatus` is used to convey the current state of the order.

If an order simultaneously exists in more than one order state, the value with the highest precedence is the value that
is reported in the `OrdStatus` field.

The order statuses are as follows (in highest to lowest precedence):

![OrderStatus_p1](OrderStatus_p1.PNG)

![OrderStatus_p2](OrderStatus_p2.PNG)

The Table below shows which state transitions have been illustrated by the matrices.

![OrderMatrix](OrderMatrix.PNG)

How to read the Order State Change Matrices:

- The row represents the current value of OrdStatus and the column represents the next value as reported back to the
  buy-side via an execution report or order cancel reject message
- Next to each OrdStatus value is its precedence – this is used when the order exists in a number of states
  simultaneously to determine the value that should be reported back
- Note that absence of a scenario should not necessarily be interpreted as meaning that the state transition is not
  allowed

### FIX 4.4 Errata

Attached is the complete [PDF]() which could be used as a reference guide for understanding different types of FIX 
messages and their order / sequences.

