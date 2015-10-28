# Session data resources

## Session data format

A "Session" object represents a sleep session. This
format is used in several resources in the API. It is later referred to as
\<SESSION DATA\>.

An example of a session object is shown below. Note that it may not always include
all of the fields. A field may be missing if it was not possible to compute it.

```javascript
{
   "id" : 1431835725,
   "start_timestamp" : 1431835725.618702,
   "end_timestamp" : 1431865087.218702,
   "timezone" : "America/New_York",
   "hardware" : "1.0/39bc22e bl:1.1 hw:1", // optional field
   "software" : "1.11.0 (101)",
   "frame_length" : 0.2, //optional field
   "error_code" : 0,
   "sampled_tracks" : { //optional field
      "normal" : {
         "samples_per_frame" : 28,
         "data_url" : "https://bedditcloud-sleepdata...", // optional field
         "data_type" : "uint16"
      },
      "noise" : {
         "samples_per_frame" : 2,
         "data_url" : "https://bedditcloud-sleepdata...", // optional field
         "data_type" : "float32"
      }
   },
   "time_value_tracks" : {
      "actigram" : {
         "value_data_type" : "float32",
         "items" : [
            [0, 29973.82],
            ...
         ]
      },
      "snoring_events" : {
         "value_data_type" : "float32",
         "items" : []
      },
      "respiration_cycles" : {
         "value_data_type" : "float32",
         "items" : [
            [40.3, 4.1],
            ...
         ]
      },
      "heart_rate" : {
         "value_data_type" : "float32",
         "items" : [
            [60, 64.59961],
            ...
         ]
      },
      "sensor_status" : {
         "value_data_type" : "uint8",
         "items" : [
            [58.4, 1],
            [29316.8, 1]
         ]
      }
   },

   // Only present in responses from server:
   "updated" : 1371482503.646541
}
```

The fields of the session object are described below.

### Id

**Together with the user id**, this id uniquely identifies the session object.
The id is derived from session_start field by truncating it into integer value.

### Timezone

The name of the timezone where the measurement was made. List of time zone names
can be found here: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

### Hardware

Sensor device version. This field is optional.

### Software

Beddit app version and build number used to create session

### Frame length

The length of the session frame in seconds. This field is optional.

### Error code

An error code describing the reason for which a Session has ended.

### Sampled tracks

Sampled tracks and its field "data_url" are optional.

#### Normal

value data type: **uint16**

Raw ballistocardiography samples measured by the Beddit sensor.

#### Noise

value data type: **float32**

The noise level, measured from 100 ms windows.

### Time-value tracks

A time-value track in a Session object is a sequence of timestamp-value pairs,
where the timestamp is a Unix timestamp (seconds since Jan 1, 1970).

#### Actigram

The timestamp represents the start time of an epoch and the value is the number of detected movements during the epoch. The movements are reported only for epochs during which the user is present on the bed. The epoch is 60 s long.

#### Snoring events

value data type: **float32**

The timestamp represents the time of a snoring event and the value is the
length of the event. The snoring events represent contiguous sequences of
respiration activity with snoring and have around 3-second resolution.

#### Respiration cycles

value data type: **float32**

The timestamp represents the start time of a respiration cycle and the value
is the length of the cycle. These respiration cycles are not suitable for
direct visualization (e.g. respiration rate curve, respiration rate variability),
but only the average respiration rate can be computed based on them.

#### Respiration cycle amplitudes

value data type = **float32**.
The timestamp represents the start time of a respiration cycle and the value is the amplitude of the cycle in arbitrary units.

#### Heart rate

value data type: **float32**

The time-value pairs represent resting heart rate readings. A new reading is
computed every 30 seconds if it can be analyzed from the signal reliably.

#### Heartbeat

value data type = **float32**

The time-value pairs represent the start timestamp and length (in seconds) of beat-to-beat intervals. There are heartbeats in the track only if they can be analyzed reliably from the signal.

#### Sensor status

value data type: **uint8**

The timestamp represent the time when the status of the sensor changes. The status
of the sensor can be 1 = operational, 2 = interference, or 3 = unclear, depending
on the existence of mains frequencies in the signal.

Value | Meaning
------|--------
1     | Operational
2     | Interference
3     | Unclear

### Updated

The updated field contains the timestamp of when the session object was put to the
server. If you want to periodically query new session objects, you can store the
latest updated value, and use it in query.

## GET /api/v1/user/:user_id/session

**Authentication**

Request must be authenticated by User specific token.

**Query parameters**

The following parameters for querying sessions are available:

Name                | Value
--------------------|----------------------------------------------
start_timestamp     | Start timestamp of range, e.g. 1371482500.646541
end_timestamp       | End timestamp of range, e.g. 1371482530.646541
updated_after       | Time of last update, e.g. 1371482503.646541

When querying you must either use the `updated_after` or both `start_timestamp` and
`end_timestamp`.

**Response**

```javascript
[
  <SESSION DATA>,
  <SESSION DATA>,
  ...
]
```
