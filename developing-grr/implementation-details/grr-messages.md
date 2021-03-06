# GRR Messages

On the wire, the client and server interchange messages. We term the
messages sent from server to the client *requests*, while messages sent
from the client to the server are *responses*. Requests sent to the
client ask the client to perform some action, for example
*ListDirectory*. We term these actions the *client action*. A single
request may elicit multiple responses.

For example, the request message *ListDirectory* will elicit a response
for each file in the directory (potentially thousands). Requests and
responses are tracked using an incremental *request\_id* and
*response\_id*.

In order to indicate when all responses have been sent for a particular
request, the client sends a special STATUS message as the last response.
In the event of errors, the message contains details of the error
(including backtrace). If the action completed successfully, an OK
status is returned.

Messages are encoded as GrrMessage protobufs:

<table>
<caption>Important GRR Message protobuf fields.</caption>
<colgroup>
<col width="50%" />
<col width="50%" />
</colgroup>
<tbody>
<tr class="odd">
<td><p>session_id</p></td>
<td><p>A unique integer for all packets generated by this flow.</p></td>
</tr>
<tr class="even">
<td><p>name</p></td>
<td><p>The name of the Action to be called on the client (See below).</p></td>
</tr>
<tr class="odd">
<td><p>args</p></td>
<td><p>A serialized protobuf which will be interpreted by the Action.</p></td>
</tr>
<tr class="even">
<td><p>request_id</p></td>
<td><p>An incrementing number of this request (see below)</p></td>
</tr>
<tr class="odd">
<td><p>response_id</p></td>
<td><p>An incrementing number of the response (see below)</p></td>
</tr>
</tbody>
</table>

![Typical Message Request/Response Sequence.](../../images/messages.png
"fig:")

Figure 2 illustrates a typical sequence of messages. Request 1 was sent
from the server to the client, and elicited 3 responses, in addition to
a status message.

When the server sends the client messages, the messages are tagged in
the data store with a lease time. If the client does not reply for these
requests within the lease time, the requests become available for lease
again. This is designed for the case of the client rebooting or losing
connectivity part way through running the action. In this case, the
request is re-transmitted and the action is run again.
