# Sleep data resources

## Sleep data format

A "Sleep" object represents the sleep measurements for a single night. This
format is used in several resources in the API. It is later referred to as
\<SLEEP DATA\>.

An example of a sleep object is shown below. Note that it may not always include
all of the fields. A field may be missing if it was not possible to compute it.

```javascript
{
  "date" : "2012-05-30",
  "timezone" : "Europe/Helsinki",
  "start_timestamp" : 1371472503.646541,
  "end_timestamp" : 1371492826.623422,
  "session_range_start" : 1371472503.646541,
  "session_range_end" : 1371492826.623422,
  "properties" : {
    "score_bed_exits" : 2,
    "score_amount_of_sleep" : 99,
    "score_snoring" : 10.3,
    "score_sleep_latency" : 5,
    "score_sleep_efficiency" : -20,
    "score_awakenings" : 1
  },
  "time_value_tracks" : {
    "actigram" : {
      "items": [
        [1393252920.32, 15],
        [1393252926.514, 9]
      ],
      "value_data_type" : "float32"
    },
    "alarm_event" : {
      "items" : [
        [1393252920.32,  0],
        [1393252926.514, 2]
      ],
      "value_data_type" : "float32"
    },
    "sleep_stages" : {
      "items" : [
        [1393249545.041, 65],
        [1393249556.041, 65]
      ],
      "value_data_type" : "int16"
    },
    "snoring_episodes" : {
      "items" : [],
      "value_data_type" : "float32"
    },
    "presence" : {
      "items" : [
        [1371472503.646541, 0]
      ],
      "value_data_type" : "float32"
    }
  },

  // Only present in responses from server:
  "updated" : 1371482503.646541
}
```

The fields of the sleep object are described below.

### Timezone

The name of the timezone where the measurement was made. List of time zone names
can be found here: http://en.wikipedia.org/wiki/List_of_tz_database_time_zones

### Properties

Scalar properties that belong to the sleep object. Generally, if a property cannot be computed, it is omitted. Also, the existence of certain properties depends on the version of the app that created the sleep.

Property | Meaning
---------|--------
sleep_time_target | The user's personal sleep time target in seconds
resting_heart_rate | The resting heart rate reading for the night
average_respiration_rate | The average respiration rate reading over the night
sleep_efficiency | The proportion of sleep in the measurement period. The time after waking up the last before ending the measurement is not incorporated in the calculation. If the subject did not fall asleep, the value is missing.
sleep_latency | The time it takes to fall asleep, in seconds. If the subject did not fall asleep, the value is missing.
away_episode_count | Number of absences during the night
total_snoring_episode_duration | Total amount of snoring, in seconds
stage_duration_A | Total time in away state in the measurement period, in seconds
stage_duration_S | Total time in sleep state in the measurement period
stage_duration_S | Total time in restless sleep state in the measurement period
stage_duration_W | Total time in wake state in the measurement period
stage_duration_N | Total amount of "no signal" time in the measurement period
stage_duration_G | Total amount of "gap" time in the measurement period
total_nap_duration | Total amount of sleep during periods marked as naps by the "nap_periods" track
sensor_status | The summarized sensor status based on the "sensor_status" track in the associated sessions.
activity_index| The number of movements per hour during measurement period. The time it takes to fall asleep and the time after waking up the last before ending the measurement are not incorporated in the calculation.
evening_HRV_index | A heart rate variability index measured during a 15 min time span during the first third of the night. The index is calculated as the root mean square of successive differences (RMSSD).
morning_HRV_index | A heart rate variability index measured during a 15 min time span during the last third of the night. The index is calculated as the RMSSD.
all_night_HRV_index | The heart rate variability index measured during the whole night. The index is calculated as the RMSSD.
resting_HRV_index | The heart rate variability index measured during a 15 min time span centered at the instance of lowest heart rate (one minute mean) during the night. The index is calculated as the RMSSD.
total_sleep_score | Sleep score calculated based on total sleep time and parameters affecting the quality of sleep. The sleep score presents a measure of the physical (not perceived) goodness of sleep during one night. The sleep score has a value between 0 and 100. In version 1 of the sleep score the value can be over 100.
sleep_score_version | The version of the sleep score. The version is an integer which is incremented whenever the sleep score algorithm changes significantly. If the property is missing, that should be interpreted as the version being 1, which is the lowest possible version. The purpose of the version is to differentiate sleep scores with different interpretations.
score_bed_exits | Sleep score item for number of bed exits
score_amount_of_sleep | Sleep score item for total amount of sleep
score_snoring | Sleep score item for total duration of snoring episodes
score_sleep_latency | Sleep score item for time it took to fall asleep
score_sleep_efficiency | Sleep score item for sleep time vs time spent in bed
score_awakenings | Sleep score item for number of awakenings during the night

### Time-value tracks

A time-value track in a Sleep object is a sequence of timestamp-value pairs,
where the timestamp is a Unix timestamp (seconds since Jan 1, 1970).
The time-value tracks are listed below.

#### Alarm event

List of alarm clock time and event type values.

Value | Meaning
------|--------
0 | Alarm started ringing
1 | User snoozed alarm
2 | User dismissed alarm
3 | Alarm was ignored (stopped automatically after x minutes)
4 | Alarm was automatically dismissed after user had ignored it several times

#### Actigram

value data type = **float32**

The timestamp represents the start time of an epoch and the value is the number of detected movements during the epoch. The movements are reported only for epochs during which the user is present on the bed. The epoch is 60 s long. This track is named activity events in sleeps created by newer versions of the app.

#### Presence value

value data type = **uint8**

Intermediate presence results. The timestamp represents the time when the subject's presence state changes during the session. The timestamp is the time when a state begins, and the value represents the subsequent presence state.

* The state 65=away represents the start time of a period when the user is away.
* The state 80=present represents the start time of the period when the user is
  present.
* The state 78=end represent the start time of the period when the presence
  of the user is unknown. This currently occurs when there is no signal.

This track is an intermediate result. To have more accurate presence data, use
the "sleep_stages" track instead.


#### Sleep stages

Value data type: **uint8**.

This is the state transition sequence for the sleep stages. The timestamp is the
time when a state begins and the value represents the new state. The gap
represents a period of missing signal in the measurement period.

Value | Meaning
------|--------
65 | Away from bed
83 | Sleep
82 | Restless sleep
87 | Awake
78 | No signal
71 | Gap in measurement

#### Snoring episodes

value data type: **float32**

The timestamp represents the time of a snoring episode and the value is the
length of the episode. The snoring episodes give a high-level view of snoring
activity and have around 10-minute resolution.

#### Sleep cycles

Value data type: **float32**

The timestamp and value represents the sleep depth at that time. The value is a
floating point number between 0.0 and 1.0. Value 0.0 corresponds to lightest
possible sleep and 1.0 to deepest possible sleep. There will be one datapoint
every 2 minutes, beginning from the moment the subject is deemed to be sleeping
and ending when the subject ultimately wakes up.

#### Heart rate curve

Value data type: **float32**

The timestamp and value pairs describe a smooth heart rate curve. The time
between curve data points is around 5 minutes. A gap in the curve is marked by a
negative value. A good way to display the curve is to split the time-value
sequence into curve parts by the negative gap-values and display each curve part
individually.

#### Nap periods
Value data type = **uint8**
This represents the starts and ends of nap periods. The value for start = 1 and for end = 2.


### Updated

The updated field contains the timestamp of when the sleep object was put to the
server. If you want to periodically query new sleep objects, you can store the
latest updated value, and use it in query.


## GET /api/v1/user/:user_id/sleep

**Authentication**

Request must be authenticated by User specific token.

**Query parameters**

The following parameters for querying sleeps are available:

Name           | Value
---------------|----------------------------------------------
start_date     | Start date of range, e.g. "2014-01-01"
end_date       | End date of range, e.g. "2014-01-02"
updated_after  | Time of last update, e.g. 1371482503.646541
limit          | Return maximum of N results, e.g. 50. Must be 1 or over.
reverse        | Return results in reverse chronological order. Allowed values "yes" or "no". Default is "no".

When querying you must either use the `updated_after` or both `start_date` and
`end_date`.

**Response**

```javascript
[
  <SLEEP DATA>,
  <SLEEP DATA>,
  ...
]
```
