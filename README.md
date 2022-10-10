# kandji2snipe
## Import/Sync Devices from Kandji to Snipe-IT
```
usage: kandji2snipe [-h] [-v] [--dryrun] [-d] [--do_not_verify_ssl] [-r] [--no_search]
                    [-u | -ui | -uf] [--mac | --iphone | --ipad | --appletv]
                    

optional arguments:
-h, --help              show this help message and exit
-v, --verbose           Sets the logging level to INFO and gives you a better
                        idea of what the script is doing.
--auto_incrementing     You can use this if you have auto-incrementing 
                        enabled in your Snipe-IT instance to utilize that 
                        instead of adding the script's default the asset tags.
--dryrun                This checks your config and tries to contact both the
                        Kandji and Snipe-IT instances, but exits before
                        updating or syncing any assets.
-d, --debug             Sets logging to include additional DEBUG messages.
--do_not_update_kandji  Does not update Kandji with the asset tags stored in
                        Snipe.
--do_not_verify_ssl     Skips SSL verification for all requests. Helpful when
                        you use self-signed certificate.
-r, --ratelimited       Puts a half second delay between Snipe-IT API calls to
                        adhere to the standard 120/minute rate limit
-f, --force             Updates the Snipe-IT asset with information from Kandji
                        every time, despite what the timestamps indicate.
-u, --users             Checks out the item to the current user in Kandji if
                        it's not already deployed
-ui, --users_inverse    Checks out the item to the current user in Kandji if
                        it's already deployed
-uf, --users_force      Checks out the item to the user specified in Kandji no
                        matter what
-uns, --users_no_search Doesn't search for any users if the specified fields
                        in Kandji and Snipe don't match. (case insensitive)
--mac                   Runs against Mac computers only
--iphone                Runs against iPhones only
--ipad                  Runs against iPads only
--appletv               Runs against Apple TVs only
```


## Overview:
This tool will sync assets between a Kandji instance and a Snipe-IT instance. The tool searches for assets based on the serial number, not the existing asset tag. If assets exist in Kandji and are not in Snipe-IT, the tool will create an asset and try to match it with an existing Snipe-IT model. This match is based on the Mac's model identifier (ex. MacBookAir7,1) being entered as the model number in Snipe-IT, rather than the model name. If a matching model isn't found, it will create one.

>Note that iPhones, iPads, and Apple TVs currently match based on the model name, rather than model identifier. All other behaviors are the same as the Macs.

When an asset is first created, it will fill out only the most basic information. When the asset already exists in your Snipe-IT inventory, the tool will sync the information you specify in the settings.conf file and make sure that the asset tag field in Kandji matches the asset tag in Snipe-IT, where Snipe-IT's info is considered the authority.

> Because it determines whether Kandji or Snipe has the most recently updated record, there is the potential to have blank data in Kandji overwrite good data in Snipe (ex. purchase date).

Lastly, if the asset tag field is blank in Kandji when the record is being created in Snipe-IT, the script will create an asset tag with KANDJI-<SERIAL NUMNER> unless you enable custom patterns in your settings.conf file.

## Requirements:

- Python3 is installed on your system with the requests, json, time, configparser, argparse, logging, datetime, and pytz python libs installed.
- Network access to both your Kandji and Snipe-IT instances.
- A [Kandji API token](https://support.kandji.io/support/solutions/articles/72000560412-kandji-api) with the following permissions:
  - Devices
    - Updated a Device
    - Device details
    - Device list
    - Device ID
- A [Snipe-IT API token](https://snipe-it.readme.io/reference#generating-api-tokens) for a user that has edit/create permissions for assets and models. 

## Installation:

### Mac

1. Install Python 3.6 or later
  - Grab the latest PKG installer from the Python website and run it.

2. Add Python 3.6 or later to your PATH
  - Run the `Update Shell Profile.command` script in the `/Applications/Python 3.X` folder to add `python3.X` your PATH

3. Create a virtualenv for kandji2snipe
  - Create the virtualenv: `mkdir ~/.virtualenv`
  - Add `python3.X` to the virtualenv: `python3.X -m venv ~/.virtualenv/kandji2snipe`
  - Activate the virtualenv: `source ~/.virtualenv/kandji2snipe/bin/activate`

4. Install dependencies
  - `pip install -r /path/to/kandji2snipe/requirements.txt`

5. Configure settings.conf (you can start by copying settings.conf.example to settings.conf)
6. Run `python kandji2snipe` & profit

### Linux

1. Copy the files to your system (recommend installing to /opt/kandji2snipe/* ). Make sure you meet all the system requirements.
2. Edit the settings.conf to match your current environment - you can start by copying settings.conf.example to settings.conf. The script will look for a valid settings.conf in /opt/kandji2snipe/settings.conf, /etc/kandji2snipe/settings.conf, or in the current folder (in that order): so either copy the file to one of those locations, or be sure that the user running the program is in the same folder as the settings.conf.

## Configuration - settings.conf: (****TO CLEAN UP****)

All of the settings that are listed in the [settings.conf.example](https://github.com/grokability/jamf2snipe/blob/main/settings.conf.example) are required except for the api-mapping section. It's recommended that you install these files to /opt/kandji2snipe/ and run them from there. You will need valid subsets from [JAMF's API](https://developer.jamf.com/apis/classic-api/index) to associate fields into Snipe-IT.

### Required

Note: do not add `""` or `''` around any values.

**[kandji]**

- `url`: https://TENANTNAME.clients.us-1.kandji.io
- `apikey`: kandji-api-bearer-token-here

**[snipe-it]**

Check out the [settings.conf.example](https://github.com/grokability/jamf2snipe/blob/main/settings.conf.example) file for the full documentation

- `url`: https://your_snipe_instance.com
- `apikey`: snipe-api-bearer-token-here
- `manufacturer_id`: The manufacturer database field id for the Apple in your Snipe-IT instance. You will probably have to create a Manufacturer in Snipe-IT and note its ID.
- `defaultStatus`: The status database field id to assign to any assets created in Snipe-IT from Kandji. Usually you will want to pick a status like "Ready To Deploy" - look up its ID in Snipe-IT and put the ID here.
- `time_zone`: The time zone that your Snipe-IT instance is set to.  Refer to APP_TIMEZONE='<timezone>' in your Snipe-IT .env file.
- `mac_model_category_id`: The ID of the category you want to assign to Mac computers. You will have to create this in Snipe-IT and note the Category ID
- `iphone_model_category_id`: The ID of the category you want to assign to iPhones. You will have to create this in Snipe-IT and note the Category ID
- `ipad_model_category_id`: The ID of the category you want to assign to iPads. You will have to create this in Snipe-IT and note the Category ID
- `appletv_model_category_id`: The ID of the category you want to assign to Apple TVs. You will have to create this in Snipe-IT and note the Category ID
    
**[asset-tag]**
- `use_custom_pattern`: Set to Yes to set your own patterns, if commented/No, devices with no asset tag will default to KANDJI-<SIERAL NUMBER>
- `pattern_mac`: The pattern to use when creating new Macs in Snipe-IT that do not have an asset tag in Kandji.
- `pattern_iphone`: The pattern to use when creating new iPhones in Snipe-IT that do not have an asset tag in Kandji.
- `pattern_ipad`: The pattern to use when creating new iPads in Snipe-IT that do not have an asset tag in Kandji.
- `pattern_appletv`: The pattern to use when creating new Apple TVs in Snipe-IT that do not have an asset tag in Kandji.

  
### API Mapping (****TO CLEAN UP****)

To get the database fields for Snipe-IT Custom Fields, go to Custom Fields, scroll down past Fieldsets to Custom Fields, click the column selection and button and select the unchecked 'DB Field' checkbox. Copy and paste the DB Field name for the Snipe under api-mapping in settings.conf.

To get the database fields for Jamf, refer to Jamf's ["Classic" API documentation](https://developer.jamf.com/apis/classic-api/index).

You need to set the manufacturer_id for Apple devices in the settings.conf file.  To get this, go to Manufacturers, click the column selection button and select the `ID` checkbox.

Some example API mappings can be found below:

- Computer Name:		`name = general name`
- MAC Address:		`_snipeit_mac_address_1 = general mac_address`
- IPv4 Address:		`_snipeit_<your_IPv4_custom_field_id> = general ip_address`
- Purchase Cost:		`purchase_cost = purchasing purchase_price`
- Purchase Date:		`purchase_date = purchasing po_date`
- OS Version:			`_snipeit_<your_OS_version_custom_field_id> = hardware os_version`
- Extension Attribute:    `_snipe_it_<your_custom_field_id> = extension_attributes <attribute id from jamf>`

More information can be found in the ./jamf2snipe file about associations and [valid subsets](https://github.com/ParadoxGuitarist/jamf2snipe/blob/master/jamf2snipe#L33).

## Testing

It is *always* a good idea to create a test environment to ensure everything works as expected before running anything in production.

Because `kandji2snipe` only ever writes the asset_tag for a matching serial number back to Kandji, testing with your production Kandji instance is OK. However, this can overwrite good data in Snipe-IT. You can spin up a Snipe-IT instance in Docker pretty quickly ([see the Snipe docs](https://snipe-it.readme.io/docs/docker)).

## Contributing

Thanks to all of the people that have already contributed to this project! If you have something you'd like to add please help by forking this project then creating a pull request to the `devel` branch. When working on new features, please try to keep existing configs running in the same manner with no changes. When possible, open up an issue and reference it when you make your pull request.
