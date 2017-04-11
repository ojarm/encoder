# Cue Insertion

This API endpoint allows SCTE-35 message injection in a frame accurate manner. Inserted commands will be reflected verbatim
in the outgonig MPEGTS streams.

If `splice` commands are inserted, they are treated as ad-break and return commands for the produced HLS and DASH streams and will create conditioned manifests.

Method | Endpoint | Action
-------|-------|----------
GET | `/scte35` | List scte35 messages
GET | `/scte35/ID` | Get a specific scte35 message
POST | `/scte35/ID` | Update or create a scte35 message
DELETE  | `/scte35/ID` | Delete a scte35 message

## Create or Update a SCTE-35 message

```bash
curl "https://<ENCODER_URL>/scte35/" \
  -X POST \
  -d type=start_break \
  -d duration=50.0 \
  -d auto_return=true \
  -d stream_time=274.033 \
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```

> The above command returns JSON structured like this:

```json
{
  "success" : true,
  "code" : 200,
  "id": "<ID>"
}
```

This endpoint creates a scte-35 message if no `<ID>` is provided. If `<ID>` is provided it updates the specified fields of the message.

### HTTP Request

`POST https://<ENCODER_URL>/scte35/<ID>`

### Fields

Parameter | Mandatory | Description
--------- | --------- | -----------
type | Yes | Type of the message. Can be `start_break`, `end_break` or `payload`.</br>`start_break` creates a `splice_insert()` command with `out_of_network_indicator` set to 1.</br>	`end_break` creates a `splice_insert()` command with `out_of_network_indicator` set to 0.</br>`payload` creates a scte-35 message from the `splice_info_section()` bytes sent by `payload` field. This way user can have full control over scte-35 message.
duration | Yes if `auto_return` is set | Duration of the break in seconds. Break duration is not inserted in the message if not provided.
auto_return | No | Sets `auto_return` flag in scte-35 message. Default is 0.
payload | Yes if `type` is equal to `payload` | Base64 encoded bytes of the `splice_info_section()` of scte-35 message.
stream_time | No | The presentation timestamp of the video frame in the input stream that the message will be inserted in seconds.
absolute_time | No | The POSIX time of the message in second plus milisecond extention.


Mandatory flag applies only if `<ID>` is not provided. If both `stream_time` and `absolute_time` is missing, scte-35 message is inserted immediately.

## List Cues

```bash
curl https://<ENCODER_URL>/scte35 \
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 163,
    "type": "start_break",
    "auto_return" : true,
    "duration" : 35.5,
    "absolute_time" : 1460648200.112
  },
  {
    "id": 164,
    "type": "payload",
    "payload": "/DA/AAAAAAAAAP/wCgUAzI3xf18AAQAAACQCGENVRUkAAAABf78BCVBvZE51bWJlcgEAAAAIQ1VFSQAAAAp99sSm",
    "stream_time" : 146.31
  }
 ]
```

This endpoint retrieves all scte-35 messages on the encoder. 

<aside class="notice">
At most 64 messages are stored and queued before execution. 
If more messages are send and the queue is full, new messages will be dropped.
</aside>

### HTTP Request

`GET https://<ENCODER_URL>/scte35`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
page_no | 0 | 0 based page number.
page_sz | 20 | Number of items in the requested page.

## Get a Specific SCTE-35 message

```bash
curl "https://<ENCODER_URL>/scte35/<ID>"
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```

> The above command returns JSON structured like this:

```json
{
    "id": 163,
    "type": "start_break",
    "auto_return" : true,
    "duration" : 35.5,
    "absolute_time" : 1460648200.112
},
```

This endpoint retrieves a scte-35 message. 

### HTTP Request

`GET https://<ENCODER_URL>/scte35/<ID>`

## Delete a Cue

```bash
curl "https://<ENCODER_URL>/scte-35/<ID>" \
  -X DELETE \
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```

> The above command returns JSON structured like this:

```json
{
  "success" : true,
  "code" : 200
}
```

This endpoint deletes a scte-35 message.

### HTTP Request

`DELETE https://<ENCODER_URL>/scte35/<ID>`


## Basic Usage

User can simply inject scte-35 messages in output stream for break start and break end by setting `type` field to `start_break` and `end_break`. Break duration and auto return flag parameters can be set in the query. You can also update an existing scte-35 message with its <ID>.

<aside class="notice">
Messages have a expiration duration of 1 hour. They will be deleted from encoder queue after 1 hour.
</aside>

> Injecting a scte-34 message for a break start. </br> Break duration will be 50 seconds and auto_return flag will be set. </br>It will be injected at 274.033 seconds of the stream.

```bash
curl "https://<ENCODER_URL>/scte35/" \
  -X POST \
  -d type=start_break \
  -d duration=50.0 \
  -d auto_return=true \
  -d stream_time=274.033 \
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```

> Injecting a scte-34 message for a break end.</br>It will be injected at 324.033 seconds of the stream.

```bash
curl "https://<ENCODER_URL>/scte35/" \
  -X POST \
  -d type=end_break \
  -d stream_time=324.033 \
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```

> Updating duration of previously sent scte-35 message with the message <ID>.

```bash
curl "https://<ENCODER_URL>/scte35/<ID>" \
  -X POST \
  -d duration=40.0 \
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```


## Injecting Arbitrary SCTE-35 Message

User can inject any arbitrary scte-35 message by setting `type` field to `payload` and sending base64 encoded bytes of the `splice_info_section()` as `payload` parameter. This way user can control every parameter and add descriptors in the message.

> Injecting an arbitrary scte-35 message

```bash
curl "https://<ENCODER_URL>/scte35/" \
  -X POST \
  -d type=payload \
  -d payload=/DA/AAAAAAAAAP/wCgUAzI3xf18AAQAAACQCGENVRUkAAAABf78BCVBvZE51bWJlcgEAAAAIQ1VFSQAAAAp99sSm \
  -d stream_time=146.31 \
  -H "Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ="
```





