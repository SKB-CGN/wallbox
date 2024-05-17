# Wallbox.com API
How to interact with the API?

In this document, i will give you a short overview, how to proceed.

**Please note:**  
This procedure only works, if you have registered an account at wallbox.com with *your own* email address. Google SSO (Single-Sign-on) will not work!

## Basic informations
```javascript
/* Basic Variables used */
let password = 'YourPassword';
let email = 'you@email.com';
let charger_id = 'idOfTheCharger';
let wallbox_token = '';
let conn_timeout = 3000;

const BASEURL = 'https://api.wall-box.com/';
const URL_AUTHENTICATION = 'auth/token/user';
const URL_CHARGER = 'v2/charger/';
const URL_CHARGER_CONTROL = 'v3/chargers/';
const URL_CHARGER_MODES = 'v4/chargers/';
const URL_CHARGER_ACTION = '/remote-action';
const URL_STATUS = 'chargers/status/';
const URL_CONFIG = 'chargers/config/';
const URL_REMOTE_ACTION = '/remote-action/';
const URL_ECO_SMART = '/eco-smart/';
```

## Token
For every request to the Wallbox.com API you need a request token. With this token, you can control the Wallbox or request data from it.
```javascript
/* Send a POST request to the API */
const options = {
    url: BASEURL + URL_AUTHENTICATION,
    timeout: conn_timeout,
    method: 'POST',
    headers: {
        'Authorization': 'Basic ' + Buffer.from(email + ":" + password).toString('base64'),
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=utf-8',
    }
};
```

The token will be returned in ``jwt `` Block of the return from the server. Create the function, to store this token inside a variable. E.g. ``wallbox_token`` as mentioned above.

With the new received token, which is valid for around 60 seconds, you can now control the Wallbox.

## Control the Wallbox
```javascript
/* Send a PUT request to the API */
const options = {
    url: BASEURL + URL_CHARGER + charger_id,
    timeout: conn_timeout,
    method: 'PUT',
    headers: {
        'Authorization': 'Bearer ' + wallbox_token,
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=utf-8',
    },
    data: JSON.stringify({
        [key]: value
    })
}
```

### Variables for key and value
| Key  | Value | Explanation |
| ------------- | ------------- | ------------- |
| `locked`  | `0 or 1`  | Unlock/Lock the Wallbox
| `maxChargingCurrent`  | `6 to 32`  | Set the current charge speed

Alternative with configs:
```javascript
/* Send a POST request to the API */
const options = {
    url: BASEURL + URL_CONFIG + charger_id,
    timeout: conn_timeout,
    method: 'POST',
    headers: {
        'Authorization': 'Bearer ' + wallbox_token,
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=utf-8',
    },
    data: JSON.stringify({
        [key 1]: value
    })
}
```

### Variables for key and value
| Key 1 | Value | Explanation | Key 2 | Value | Explanation |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| `energy_price`  | float number  | price per kWh | `currency_id` | idx 1 to n | Optional the currency index, 1 for euro
| `max_charging_current`  | `6 to 32`  | Set the current charge speed | NO KEY2 | | |
| `home_sharing` | `0 or 1` | disable or enable power boost | `icp_max_current` | `1 to 64 or -1 to -64` | set the max power per phase use positive if `home_sharing` is 1 , negative if 0

```javascript
/* Send a POST request to the API */
const options = {
    url: BASEURL + URL_CHARGER_CONTROL + charger_id + URL_REMOTE_ACTION,
    timeout: conn_timeout,
    method: 'POST',
    headers: {
        'Authorization': 'Bearer ' + wallbox_token,
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=utf-8',
    },
    body: JSON.stringify({
        'action': value
    })
}
```
### Variables for action value
| Value | Explanation |
|  ------------- | ------------- |
|  `1` | Resume - Mode
|  `2` | Pause - Mode
|  `3` | Reboot the Wallbox
|  `4` | Factory - Rest the Wallbox. ! Be careful !
|  `5` | Install Software Update if available
|  `9` | After a manual stop: resume schedule and ecosmart mode


## Control Wallbox Modes
```javascript
/* Send a PUT request to the API */
const options = {
    url: BASEURL + URL_CHARGER_MODES + charger_id + URL_ECO_SMART,
    timeout: conn_timeout,
    method: 'PUT',
    headers: {
        'Authorization': 'Bearer ' + wallbox_token,
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=utf-8',
    },
    data: JSON.stringify({
        "data" : {
             "attributes": {
                    "percentage": 100,
                    "enabled": 0,
                     "mode": 0
            },
            "type": "eco_smart"
        }
    })
}
```
| Key  | Value | Explanation |
| ------------- | ------------- | ------------- |
| `enabled` | `0 or 1` | enable or disable eco smart
| `mode` | `0 or 1` | 0 is Eco Mode, 1 is Full Green 

## Receive Data from the Wallbox - API
There are 2 information adresses, which provide informations. One is "basic" information about the Wallbox itself and some diagnosis. The other one - i call it "extended", delivers informations about about charging and past sessions.

### Basic information
```javascript
/* Send a PUT request to the API */
const options = {
    url: BASEURL + URL_CHARGER + charger_id,
    timeout: conn_timeout,
    method: 'PUT',
    headers: {
        'Authorization': 'Bearer ' + token,
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=utf-8',
    }
}
```

You will receive a json, with basic informations and the current state of the Wallbox inside ``status``.

```json
{
    "data": {
        "chargerData": {
            "id": 00000,
            "uid": "xyz",
            "uniqueIdentifier": "wsUnanQQ",
            "serialNumber": "xxwweerrr",
            "name": "PulsarPlus SN 00000",
            "group": 289814,
            "groupUid": "xyz",
            "chargerType": "PulsarPlus",
            "softwareVersion": "5.17.59",
            "softwareUpdatedAt": 1689715588,
            "status": 210,
            "ocppConnectionStatus": 4,
            "ocppReady": "ocpp_1.6j",
            "stateOfCharge": null,
            "maxChgCurrent": null,
            "maxAvailableCurrent": 32,
            "maxChargingCurrent": 32,
            "locked": 1,
            "lastConnection": 1692341408,
            "lastSync": {
                "date": "2023-08-17 20:01:46.000000",
                "timezone_type": 3,
                "timezone": "UTC"
            },
            ...
            "resume": {
                "totalUsers": 1,
                ...
            }
        },
        "users": [
            {
                "id": 286113,
                ...
            }
        ]
    }
}
```

### Extended Information
```javascript
/* Send a GET request to the API */
const options = {
    url: BASEURL + URL_STATUS + charger_id,
    timeout: conn_timeout,
    method: 'GET',
    headers: {
        'Authorization': 'Bearer ' + token,
        'Accept': 'application/json, text/plain, */*',
        'Content-Type': 'application/json;charset=utf-8',
    }
}
```
You will receive a json, with extended and more informations, which is around 200 lines long. The most interesting JSON property here would be ``charging_power`` and ``status_id`` - explained below.

```json
{
    "user_id": 286113,
    "user_name": "Stephan",
    "car_id": 1,
    "car_plate": "",
    "depot_price": 0.2,
    "last_sync": "2023-08-18 06:50:26",
    "power_sharing_status": 0,
    "mid_status": 1,
    "status_id": 210,
    "name": "PulsarPlus SN 9988552",
    "charging_power": 0,
    "max_available_power": 32,
    "depot_name": "Kreyenborg.koeln",
    "charging_speed": 0,
    "added_range": 4,
    "added_energy": 0.546,
    "added_green_energy": 0,
    "added_discharged_energy": 0,
    "added_grid_energy": 0,
    "charging_time": 585,
    "finished": true,
    "cost": 0,
    "current_mode": 3,
    "preventive_discharge": false,
    "state_of_charge": null,
    "ocpp_status": 4,
    "config_data": {
        "charger_id": 9988552,
        "uid": "xys",
        "serial_number": "9988552",
        "name": "PulsarPlus SN 9988552",
        "locked": 1,
        "auto_lock": 1,
        "auto_lock_time": 300,
        "multiuser": 0,
        "max_charging_current": 32,
        "language": "EN",
        "icp_max_current": 0,
        "grid_type": 1,
        "energy_price": 0.24,
        "energyCost": {
            "value": 0.24,
            "inheritedGroupId": null
        },
        "unlock_user_id": 286113,
        "power_sharing_config": 256,
        "purchased_power": 0,
        "show_name": 1,
        "show_lastname": 1,
        "show_email": 1,
        "show_profile": 1,
        "show_default_user": 1,
        "gesture_status": 7,
        "home_sharing": 0,
        "dca_status": 0,
        "connection_type": 1,
        "max_available_current": 32,
        "live_refresh_time": 30,
        "update_refresh_time": 300,
        "owner_id": 286113,
        "remote_action": 0,
        "rfid_type": null,
        "charger_has_image": 0,
        "sha256_charger_image": null,
        "plan": {
            "plan_name": "Business",
            "features": [
                "DEFAULT_FEATURE",
                "POWER_BOOST",
                "MOBILE_CONNECTIVITY",
                "CHARGER_SUBGROUPS",
                "USER_SUBGROUPS",
                "DYNAMIC_POWER_SHARING",
                "PAYMENTS",
                "SET_UP_INCLUDED",
                "BULK_ACTIONS",
                "AUTOMATIC_REPORTING",
                "BILLING",
                "STATISTICS"
            ]
        },
        "sync_timestamp": 1692301922,
        "currency": {
            "id": 1,
            "name": "Euro Member Countries",
            "symbol": "€",
            "code": "EUR"
        },
        "charger_load_type": "Private",
        "contract_charging_available": false,
        "country": {
            "id": 4,
            "code": "DEU",
            "iso2": "DE",
            "name": "ALEMANIA",
            "phone_code": "49"
        },
        "state": null,
        "timezone": "Europe/Berlin",
        "part_number": "PLP1-M-2-4-9-002-F",
        "software": {
            "updateAvailable": false,
            "currentVersion": "5.17.59",
            "latestVersion": "5.17.59",
            "fileName": "plp1_5.17.59.tar"
        },
        "available": 1,
        "operation_mode": "ocpp",
        "ocpp_ready": "ocpp_1.6j",
        "tariffs": [],
        "mid_enabled": 0,
        "mid_margin": 1,
        "mid_margin_unit": 1,
        "mid_serial_number": "",
        "mid_status": 1,
        "session_segment_length": 0,
        "group_id": 289814,
        "user_socket_locking": 0,
        "sim_iccid": null,
        "ecosmart": {
            "enabled": false,
            "mode": 0,
            "percentage": 100
        }
    }
}
```
### Status Informations inside both JSON properties `status`
| Status Code  | Explanation |
| ------------- | ------------- |
0| Disconnected|
|14| Error|
|15| Error|
|161| Ready|
|162| Ready|
|163| Disconnected|
|164| Waiting|
|165| Locked|
|166| Updating|
|177| Scheduled|
|178| Paused|
|179| Scheduled|
|180| Waiting for car demand|
|181| Waiting for car demand|
|182| Paused|
|183| Waiting in queue by Power Sharing|
|184| Waiting in queue by Power Sharing|
|185| Waiting in queue by Power Boost|
|186| Waiting in queue by Power Boost|
|187| Waiting MID failed|
|188| Waiting MID safety margin exceeded|
|189| Waiting in queue by Eco-Smart|
|193| Charging|
|194| Charging|
|195| Charging|
|196| Discharging|
|209| Locked|
|210| Locked - Car connected
