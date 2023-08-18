# Wallbox.com API
How to interact with the API in Javascript?

In this document, i will give you a short overview, how to proceed.

## Basic informations
```
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
You will receive a json, with extended and more informations, which is around 200 lines long. The most intersting JSON property here would be ``charging_power``.

