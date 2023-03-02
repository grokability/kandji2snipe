# kandji2snipe

## About

This `python3` script leverages the [Kandji](https://kandji.io) and [Snipe-IT](https://snipeitapp.com) APIs to accomplish two things:
* Sync device details from Kandji to Snipe-IT
* Sync asset tags from Snipe-IT to Kandji.

## Overview:
kandji2snipe is designed to sync assets between your Kandji and Snipe-IT instances. The tool searches for assets based on the serial number, not the existing asset tag. If assets exist in Kandji and are not in Snipe-IT, the tool will create an asset and try to match it with an existing Snipe-IT model. This match is based on the device's model identifier (ex. MacBookAir7,1) being entered as the model number in Snipe-IT, rather than the model name. If a matching model isn't found, it will create one.

When an asset is first created, it will fill out only the most basic information. If the asset already exists in your Snipe-IT inventory, the tool will sync the information you specify in the settings.conf file and make sure that the asset tag in Kandji matches the asset tag in Snipe-IT, where Snipe-IT's asset tag for that device is considered the authority.

If the asset tag field is blank in Kandji when the record is being created in Snipe-IT, the script will create an asset tag with `KANDJI-$SERIAL_NUMBER` unless you enable `use_custom_pattern` in your `settings.conf` file.

## Requirements:

- Python3
- Python dependencies - Can be installed using the included `requirements.txt` using the command `python3 -m pip install -r requirements.txt` or individually using the commands `python3 -m pip install requests` and `python3 -m pip install pytz`.
- A [Kandji API token](https://support.kandji.io/support/solutions/articles/72000560412-kandji-api) with the following permissions:
  - Devices
    - Updated a Device
    - Device details
    - Device list
    - Device ID
- A [Snipe-IT API key](https://snipe-it.readme.io/reference#generating-api-tokens) for a user with the following permissions:
  - Assets
    - View
    - Create
    - Edit
    - Checkin
    - Checkout
  - Users
    - View
  - Models
    - View
    - Create
    - Edit
  - Self
    - Two-Factor Authentication *(recommended, but not required)*
    - Create API Keys

## Configuration - settings.conf:

All of the keys highlighted in **bold** below are *required* in your `settings.conf` file. Please review the [settings.conf.example](https://github.com/briangoldstein/kandji2snipe/blob/main/settings.conf.example) file for example settings and additional details.

Note: do not add `""` or `''` around any values.

**[kandji]**

- **`tenant`**: Your tenant name, for example accuhive.kandji.io would be **accuhive**
- **`region`**: **us** or **eu**
- **`apitoken`**: Your [Kandji API token](https://support.kandji.io/support/solutions/articles/72000560412-kandji-api)

**[snipe-it]**

- **`url`**: The base URL for your Snipe-IT instance (ie https://snipeit.example.com)
- **`apikey`**: Your [Snipe-IT API key](https://snipe-it.readme.io/reference#generating-api-tokens)
- **`time_zone`**: The time zone that your Snipe-IT instance is set to.  Refer to `APP_TIMEZONE=` in your Snipe-IT .env file.
- **`manufacturer_id`**: The manufacturer database field id for the Apple in your Snipe-IT instance.
- **`defaultStatus`**: The status database field id to assign to new assets created in Snipe-IT from Kandji. Typically you will want to pick a status like "Ready To Deploy".
- **`mac_model_category_id`**: The ID of the category you want to assign to Mac computers. You will have to create this in Snipe-IT and note the Category ID.
- **`iphone_model_category_id`**: The ID of the category you want to assign to iPhones. You will have to create this in Snipe-IT and note the Category ID.
- **`ipad_model_category_id`**: The ID of the category you want to assign to iPads. You will have to create this in Snipe-IT and note the Category ID.
- **`appletv_model_category_id`**: The ID of the category you want to assign to Apple TVs. You will have to create this in Snipe-IT and note the Category ID.
- `mac_custom_fieldset_id`: The ID of the category you want to assign to Mac computers. You will have to create this in Snipe-IT and note the Category ID.
- `iphone_custom_fieldset_id`: The ID of the category you want to assign to iPhones. You will have to create this in Snipe-IT and note the Category ID.
- `ipad_custom_fieldset_id`: The ID of the category you want to assign to iPads. You will have to create this in Snipe-IT and note the Category ID.
- `appletv_custom_fieldset_id`: The ID of the category you want to assign to Apple TVs. You will have to create this in Snipe-IT and note the Category ID.


**[asset-tag]**
- **`use_custom_pattern`**: Set to **yes** to set your own patterns, if set to **no**, devices with no existing asset tag in Kandji will default to `KANDJI-$SIERAL_NUMBER`.
- `pattern_mac`: The pattern to use when creating new Macs in Snipe-IT that do not have an asset tag in Kandji.
- `pattern_iphone`: The pattern to use when creating new iPhones in Snipe-IT that do not have an asset tag in Kandji.
- `pattern_ipad`: The pattern to use when creating new iPads in Snipe-IT that do not have an asset tag in Kandji.
- `pattern_appletv`: The pattern to use when creating new Apple TVs in Snipe-IT that do not have an asset tag in Kandji.

**[mac-api-mapping]**

**[iphone-api-mapping]**

**[ipad-api-mapping]**

**[appletv-api-mapping]**

  
### API Mapping

To get the database fields for Snipe-IT Custom Fields, go to Settings and then Custom Fields inside of your Snipe-IT instance, scroll down past Fieldsets to Custom Fields, click the column selection button and make sure the 'DB Field' checkbox is checked. Copy and paste the DB Field name for Snipe-IT under *platform*-api-mapping sections in your `settings.conf` file.

To get the API mapping fields for Kandji, refer to Kandji's [Device Details](https://api-docs.kandji.io/#e320f334-d7d4-4c15-b75d-bc956b2943d5) API example response.

You need to set the `manufacturer_id` for Apple devices in the `settings.conf` file. You can grab the `manufacturer_id` in Snipe-IT by going to Manufacturers > click the column selection button > select the `ID` checkbox.

Some example API mappings can be found below:

- Computer Name:		`name = general name`
- MAC Address:		`_snipeit_<your_mac_address_custom_field_id> = general mac_address`
- IPv4 Address:		`_snipeit_<your_IPv4_custom_field_id> = general ip_address`
- Blueprint Name:		`_snipeit_<your_blueprint_custom_field_id>_ = general blueprint_name`
- FileVault Status:		`_snipeit_<your_filevault_enabled_custom_field_id> = filevault filevault_enabled`
- Kandji Device ID:			`_snipeit_<your_device_id_custom_field_id> = general device_id`


## Usage
```
usage: kandji2snipe [-h] [-l] [-v] [-d] [--dryrun] [--version] [--auto_incrementing] [--do_not_update_kandji] [--do_not_verify_ssl] [-r] [-f] [-u] [-uns] [--mac | --iphone | --ipad | --appletv]

optional arguments:
  -h, --help              Shows this help message and exits.
  -l, --logfile           Saves logging messages to kandji2snipe.log instead of displaying on screen.
  -v, --verbose           Sets the logging level to INFO and gives you a better idea of what the script is doing.
  -d, --debug             Sets logging to include additional DEBUG messages.
  --dryrun                This checks your config and tries to contact both the Kandji and Snipe-IT instances, but exits before updating or syncing any assets.
  --version               Shows the version of this script and exits.
  --auto_incrementing     You can use this if you have auto-incrementing enabled in your Snipe-IT instance to utilize that instead of using KANDJI-<SERIAL NUMBER> for the asset tag.
  --do_not_update_kandji  Does not update Kandji with the asset tags stored in Snipe-IT.
  --do_not_verify_ssl     Skips SSL verification for all Snipe-IT requests. Helpful when you use self-signed certificate.
  -r, --ratelimited       Puts a half second delay between Snipe-IT API calls to adhere to the standard 120/minute rate limit
  -f, --force             Updates the Snipe-IT asset with information from Kandji every time, despite what the timestamps indicate.
  -u, --users             Checks in/out assets based on the user assignment in Kandji.
  -uns, --users_no_search Doesn't search for any users if the specified fields in Kandji and Snipe-IT don't match. (case insensitive)
  --mac                   Runs against Kandji Mac computers only.
  --iphone                Runs against Kandji iPhones only.
  --ipad                  Runs against Kandji iPads only.
  --appletv               Runs against Kandji Apple TVs only.

```

## Testing

It is recommended that you use a test/dev Snipe-IT instance for testing and that your backups are up to date. You can spin up a Snipe-IT instance in Docker pretty quickly ([see the Snipe-IT docs](https://snipe-it.readme.io/docs/docker)).

If you do not have a test/dev Kandji tenant, you can test kandji2snipe using the `--do_not_update_kandji` argument to prevent data being written back to Kandji.

## Acknowledgements

**kandji2snipe** is inspired by and forked from [jamf2snipe](https://github.com/grokability/jamf2snipe), created by [Brian Monroe](https://github.com/ParadoxGuitarist). Thank you for your contributions to the Mac Admin and Open Source communities!

## Contributing

If you have something you'd like to add please help by forking this project then creating a pull request to the `devel` branch. When working on new features, please try to keep existing configs running in the same manner with no changes. When possible, open up an issue and reference it when you make your pull request.
