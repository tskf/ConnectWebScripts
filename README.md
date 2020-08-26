# Connect Web Scripts

Use in web-browser console (`F12` key).

* [Download all activities](#download-all-activities)
* [Change activity's access](#change-activitys-access)
* [Change activity's type](#change-activitys-type)
* [Search activities without gear](#search-activities-without-gear)
* [Search activities by course](#search-activities-by-course)
* [Search activities by device](#search-activities-by-device)
* [Sum distance by device](#sum-distance-by-device)
* [Delete activity request](#delete-activity-request)
* [Filters for the Segments' Map](#filters-for-the-segments-map)


## Search parameters
* It's normal search link from Connect Web, if you need more filters, check them on the website and add by `&`.
* Set the `limit` to match your activities count.
```
?limit= number of activities to change, starting from latest
&start= start from nth+1 activity, counting from latest
&activityType= running, cycling, fitness_equipment, walking, other
&startDate= YEAR-MM-DD
&endDate= YEAR-MM-DD
&minDistance= in meters
&maxDistance= in meters
```

### Example
```javascript
.../search/activities?limit=200&start=200&activityType=running
```


## Download all activities

* When downloading larger files, set more wait time: `t+=10000`.
* When a lot of files, use the `start` parameter to download in batches.
* Download 1 `GPX` file by hand, and click on the checkbox to download all `GPX` files without asking.
* Open the console and check `Persist Logs` (to keep logs about all files).

```javascript
jQuery.getJSON(
    'https://connect.garmin.com/modern/proxy/activitylist-service/activities/search/activities?limit=200',
    function(act_list)
    {
        var t = 0;
        act_list.forEach(
        function(act)
            {
                setTimeout(function() {
                    console.dir(act['activityId'], act['activityName'], act['startTimeLocal']);
                    location = 'https://connect.garmin.com/modern/proxy/download-service/export/gpx/activity/' + act['activityId'];
                }, t+=5000);
            }
        );
    }
);
```

* FIT or GPX or TCX:
```
.../download-service/files/activity/
.../download-service/export/gpx/activity/
.../download-service/export/tcx/activity/
```

## Change activity's access

* Set `typeKey` to privacy level you need.
* `limit=1` for testing, it will change only first latest activity to `private`.

```javascript
jQuery.getJSON(
    'https://connect.garmin.com/modern/proxy/activitylist-service/activities/search/activities?limit=1',
    function(act_list)
    {
        act_list.forEach(
        function(act)
            {
                fetch('https://connect.garmin.com/modern/proxy/activity-service/activity/' + act['activityId'],
                {
                    'headers': { 'Content-Type': 'application/json', 'NK': 'NT', 'X-HTTP-Method-Override': 'PUT' },

                    'body': '{"accessControlRuleDTO":{"typeKey":"private"},"activityId":' + act['activityId'] + '}',

                    'method': 'POST'
                });
                console.dir(act['activityId'], act['activityName'], act['startTimeLocal']);
            }
        );
    }
);
```

### Parameters
```
typeKey: private, subscribers, groups, public
```


## Change activity's type

* Set `typeKey` to a type you need.
* `limit=1` for testing, it will change only first latest activity to `running`.

```javascript
jQuery.getJSON(
    'https://connect.garmin.com/modern/proxy/activitylist-service/activities/search/activities?limit=1',
    function(act_list)
    {
        act_list.forEach(
        function(act)
            {
                fetch('https://connect.garmin.com/modern/proxy/activity-service/activity/' + act['activityId'],
                {
                    'headers': { 'Content-Type': 'application/json', 'NK': 'NT', 'X-HTTP-Method-Override': 'PUT' },

                    'body': '{"activityTypeDTO":{"typeId":6,"typeKey":"running","parentTypeId":1,"sortOrder":6},"activityId":' + act['activityId'] + '}',

                    'method': 'POST'
                });
                console.dir(act['activityId'], act['activityName'], act['startTimeLocal']);
            }
        );
    }
);
```


## Search activities without gear

```javascript
jQuery.getJSON(
    'https://connect.garmin.com/modern/proxy/activitylist-service/activities/search/activities?limit=200',
    function(act_list)
    {
        act_list.forEach(
        function(act)
            {
                jQuery.getJSON(
                    'https://connect.garmin.com/modern/proxy/gear-service/gear/filterGear?activityId=' + act['activityId'],
                    function(gear)
                        {
                            if(gear.length === 0)
                            {
                                console.dir(act['activityId'], act['activityName'], act['startTimeLocal']);
                            }
                        }
                );
            }
        );
    }
);
```


## Search activities by course

* Replace `!!!COURSE_ID!!!` by your course id (a number from link).

```javascript
jQuery.getJSON(
    'https://connect.garmin.com/modern/proxy/activitylist-service/activities/search/activities?limit=200',
    function(act_list)
    {
        act_list.forEach(
        function(act)
            {
                jQuery.getJSON(
                    'https://connect.garmin.com/modern/proxy/activity-service/activity/' + act['activityId'],
                    function(act_details)
                        {
                            if(act_details['metadataDTO']['associatedCourseId'] === !!!COURSE_ID!!!)
                            {
                                console.dir(act['activityId'], act['activityName'], act['startTimeLocal'], act['distance'], (act['duration']/60).toFixed(2)+' minutes');
                            }
                        }
                );
            }
        );
    }
);
```


## Search activities by device

* Replace `!!!DEVICE_ID!!!` by your device id (copy a number from a link to settings for that device).

```javascript
jQuery.getJSON(
    'https://connect.garmin.com/modern/proxy/activitylist-service/activities/search/activities?limit=200',
    function(act_list)
    {
        act_list.forEach(
        function(act)
            {
                if(act['deviceId'] === !!!DEVICE_ID!!!)
                {
                    console.dir(act['activityId'], act['activityName'], act['startTimeLocal'], act['distance'].toFixed());
                }
            }
        );
    }
);
```


## Sum distance by device

* In meters.
* Replace `!!!DEVICE_ID!!!` by your device id (a number from a link to settings for that device).

```javascript
jQuery.getJSON(
    'https://connect.garmin.com/modern/proxy/activitylist-service/activities/search/activities?limit=200',
    function(act_list)
    {
        var distance = 0;
        act_list.forEach(
        function(act)
            {
                if(act['deviceId'] === !!!DEVICE_ID!!!)
                {
                    console.dir(act['activityId'], act['activityName'], act['startTimeLocal'], act['distance']);
                    distance += act['distance'];
                }
            }
        );
        console.dir('Sum:', distance);
    }
);
```


## Delete activity request

```javascript
fetch('https://connect.garmin.com/modern/proxy/activity-service/activity/' + act['activityId'],
{
    "headers": { 'Accept': 'application/json', 'NK': 'NT', "X-HTTP-Method-Override": "DELETE" },
    "method": "POST"
});
```


## Filters for the Segments' Map

The Segments' Map resets every time you open it...<br>
To make it easier, save bookmark with javascript. It needs to be a one line.

### Example
Set city: `CityName` and Activity Type: `Running`:
```javascript
javascript:{var d=document; d.getElementsByTagName("input")[0].value="CityName"; d.getElementsByName("1")[1].click(); d.getElementsByClassName("icon-search")[0].click();}
```

### Other filters
Change or add to the javascript above:

Location, change `CityName` to yours:<br>
`d.getElementsByTagName("input")[0].value="CityName";`

Activity Type: `[0]`-any, `[1]`-running, `[2]`-cycling:<br>
`d.getElementsByName("1")[1].click();`

Segment Type: `[0]`-any, `[1]`-sprint, `[2]`-hills, `[3]`-downhill, `[4]`-hill, `[5]`-other:<br>
`d.getElementsByName("2")[5].click();`

Surface: `[0]`-any, `[1]`-gravel/unpaved, `[2]`-dirt, `[3]`-mixed, `[4]`-paved:<br>
`d.getElementsByName("3")[4].click();`

This will click search, activating filters, must be last:<br>
`d.getElementsByClassName("icon-search")[0].click();`