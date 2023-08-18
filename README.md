# Wallbox.com API
How to interact with the API?

In this document, i will give you a short overview, how to proceed.

## Basic informations
```
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
const URL_CHARGER_ACTION = '/remote-action';
const URL_STATUS = 'chargers/status/';
```

## Token
For every request to the Wallbox.com API you need a request token. With this token, you can control the Wallbox or request data from it.
```
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
```
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
    body: JSON.stringify({
        [key]: value
    })
}
```

### Variables for key and value
| Key  | Value | Explanation |
| ------------- | ------------- | ------------- |
| `locked`  | `0 or 1`  | Unlock/Lock the Wallbox
| `maxChargingCurrent`  | `6 to 32`  | Set the current charge speed
| `action` | `1` | Resume - Mode
| `action` | `2` | Pause - Mode
| `action` | `3` | Reboot the Wallbox
| `action` | `4` | Factory - Rest the Wallbox. ! Be careful !
| `action` | `5` | Install Software Update if available

## Receive Data from the Wallbox - API
There are 2 information adresses, which provide informations. One is "basic" information about the Wallbox itself and some diagnosis. The other one - i call it "extended", delivers informations about about charging and past sessions.

### Basic information
```
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

You will receive a json, with basic informations.

```
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
```
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
You will receive a json, with extended and more informations, which is around 200 lines long. The most interesting JSON property here would be ``charging_power``.

```
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
