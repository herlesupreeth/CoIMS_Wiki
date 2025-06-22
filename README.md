# CoIMS (Carrier Config overriding IMS settings)
Guide for overriding IMS settings to force enable VoLTE/VoWiFi using Carrier Privileges

## Requirements
- A programmable version of USIM/ISIM with KIC1, KID1 and KIK1, or *a non-programmable USIM/ISIM with ARA-M application but with option to push certficates to ARA-M via OTA*
- VoLTE/VoWiFi capable phone with Android Oreo (8.0) or above
- PCSC, serial card reader (SIM card programmer)
- Java v1.8
- Lastest pySim software - https://github.com/osmocom/pysim
	- Manual - https://downloads.osmocom.org/docs/pysim/master/osmopysim-usermanual.pdf

## My Setup
- sysmoUSIM-SJS1-4ff USIM with ADM keys (supports JavaCard 2.2.1 only)
- sysmoISIM-SJA2 USIM + ISIM with ADM keys
- OnePlus 5t UE with Android Pie
- Gemalto SIM programmer

## Big shout out and credits to following people for their awesome work
<p align="justify">
	<a href="https://github.com/martinpaljak">Martin Paljak</a> for GlobalPlatformPro (gp.jar) - A tool to load and manage applets on compatible JavaCards from command line.
</p>

<p align="justify">
	<a href="https://github.com/bertrandmartel">Bertrand Martel</a> for ARA-M applet (applet.cap) - ARA-M implementation for JavaCards. ARA-M is an application (typically present on a SIM card) which manage access rules that are enforced by an Access Control Enforcer (typically present on Android device). The enforcer makes sure the rules from the ARAM are enforced. An access rule is composed of an AID, a certificate hash (SHA1/SHA256 of client application cert) and a set of rules. The Access Control enforcer will allow/deny a client application (for example an Android app) to send APDU to a Secure Element (SE) applet based on these rules.
</p>

<p align="justify">
	<a href="https://github.com/osmocom/pysim">Entire pySim team</a> for providing a feature rich software to program anything and everything on a SIM card.
</p>

> **⚠️ Warning:** While using latest Sysmocom SIM cards (e.g. sysmoISIM-SJA5), the ARA-M applet is already pre-provisioned. You can skip directly to **Step 4** below.

## Steps

#### Step 1: Clone repository and fetch details of the SIM

<p align="justify">
	In order to install and/or manage Java Card applets on your SIM card, make sure to have KIC1, KID1 and KIK1 keys. KIC1, KID1 and KIK1 could differ from one SIM card to another so make sure to have the correct keys. <i>If you have a non-programmable USIM/ISIM with ARA-M application and have option to push certficates to ARA-M via OTA, jump to Step 4</i>
</p>

```
$ git clone https://github.com/herlesupreeth/CoIMS_Wiki
$ cd CoIMS_Wiki
$ alias gp="java -jar $PWD/gp.jar"
```

Example: In sysmoUSIM-SJS1-4ff USIM cards, the key mappings for GlobalPlatformPro are as follows

sysmoUSIM key | GlobalPlatformPro argument
------------- | --------------------------
KIC1 | --key-enc
KID1 | --key-mac
KIK1 | --key-dek

<p align="justify">
	Fetch details of the SIM by replacing KIC1, KID1 and KIK1 with correct keys respective to your SIM card. Execution of below command should not result in any error. If there is an error, please check the error and double check everything before proceeding
</p>

```
$ gp --key-enc <KIC1> --key-mac <KID1> --key-dek <KIK1> -lvi
```

#### Step 2: Unlock the SIM card for easier installation of applet as follows (Optional)

**Proceed with caution when unlocking SIM card as it could brick your USIM/ISIM if incorrect KIC1, KID1 and KIK1 keys are used**

```
$ gp --key-enc <KIC1> --key-mac <KID1> --key-dek <KIK1> --unlock
```

Example: A sysmoUSIM-SJS1-4ff USIM card with following keys is unlocked as follows

KIC1 = --key-enc = 975B496CED1F2FB984145A55AB31A585

KID1 = --key-mac = E7207B567F9D08726A6EFBD90C50DA9A

KIK1 = --key-dek = DEAA4E9A9B3BC6FC5EFF77A8E9925632

```
$ gp --key-enc 975B496CED1F2FB984145A55AB31A585 --key-mac E7207B567F9D08726A6EFBD90C50DA9A --key-dek DEAA4E9A9B3BC6FC5EFF77A8E9925632 --unlock
Default type=DES3 bytes=404142434445464748494A4B4C4D4E4F kcv=8BAF47 set as master key for A000000003000000
```

#### Step 3: Install ARA-M Java Card applets on USIM/ISIM

**Proceed with caution when installing applets on SIM card as it could brick your USIM/ISIM if incorrect KIC1, KID1 and KIK1 keys are used**

Install the ARA-M applet (applet.cap). The following command must execute without any errors.

```
# If SIM is not unlocked in Step 2
$ gp --key-enc <KIC1> --key-mac <KID1> --key-dek <KIK1> --install applet.cap
# If SIM is unlocked in Step 2
$ gp --install applet.cap
```

#### Step 4: Push the SHA-1 certifcate of the Carrier Config Android app onto ARA-M in USIM/ISIM

The Carrier Config Android app which will be installed in Step 5 is signed with following SHA1 key

SHA1: E4:68:72:F2:8B:35:0B:7E:1F:14:0D:E5:35:C2:A8:D5:80:4F:0B:E3

In order to provide Carrier Privileges to Carrier Config app, push the above SHA1 certifcate as follows

```
# If SIM is not unlocked in Step 2
$ gp --key-enc <KIC1> --key-mac <KID1> --key-dek <KIK1> -a 00A4040009A00000015141434C0000 -a 80E2900033F031E22FE11E4F06FFFFFFFFFFFFC114E46872F28B350B7E1F140DE535C2A8D5804F0BE3E30DD00101DB080000000000000001
# If SIM is unlocked in Step 2
$ gp -a 00A4040009A00000015141434C0000 -a 80E2900033F031E22FE11E4F06FFFFFFFFFFFFC114E46872F28B350B7E1F140DE535C2A8D5804F0BE3E30DD00101DB080000000000000001
```

The split-up of above APDU sent to SIM card is as follows

<pre>
#### REF-AR-DO for UICC Carrier Privileges

|REF-AR-DO|T|E2    | |                  | |                                        |
|         |L|2F    | |                  | |                                        |
|         |V|REF-DO|T|E1                | |                                        |
|         | |      |L|1E                | |                                        |
|         | |      |V|AID-REF-DO        |T|4F                                      |
|         | |      | |                  |L|06                                      |
|         | |      | |                  |V|FFFFFFFFFFFF                            |
|         | |      | |DeviceAppID-REF-DO|T|C1                                      |
|         | |      | |                  |L|14                                      |
|         | |      | |                  |V|E46872F28B350B7E1F140DE535C2A8D5804F0BE3|
|         | |AR-DO |T|E3                | |                                        |
|         | |      |L|0D                | |                                        |
|         | |      |V|APDU-AR-DO        |T|D0                                      |
|         | |      | |                  |L|01                                      |
|         | |      | |                  |V|01 (Always)                             |
|         | |      | |PERM-AR-DO        |T|DB                                      |
|         | |      | |                  |L|08                                      |
|         | |      | |                  |V|0000000000000001                        |
</pre>

**Alternatively, pySim-Shell can be used to push the certificate onto ARA-M applet**

```
pySIM-shell (00:MF)> select ADF.ARA-M
null
pySIM-shell (00:MF/ADF.ARA-M)> aram_store_ref_ar_do --aid FFFFFFFFFFFF --device-app-id E46872F28B350B7E1F140DE535C2A8D5804F0BE3 --apdu-always --android-permissions 0000000000000001
```

#### Step 5: To check the list of installed certificates

```
# If SIM is not unlocked in Step 2
$ gp --key-enc <KIC1> --key-mac <KID1> --key-dek <KIK1> --acr-list-aram
RULE #0 :
       AID  : FFFFFFFFFFFF
       HASH : E46872F28B350B7E1F140DE535C2A8D5804F0BE3
       APDU rule   : ALWAYS(0x01)
# If SIM is unlocked in Step 2
$ gp --acr-list-aram
RULE #0 :
       AID  : FFFFFFFFFFFF
       HASH : E46872F28B350B7E1F140DE535C2A8D5804F0BE3
       APDU rule   : ALWAYS(0x01)
```

<p align="justify">
	<i>If you have a non-programmable USIM/ISIM with ARA-M application and have option to push certficates to ARA-M via OTA, push the above SHA1 certificate on to the SIM</i>
</p>

**Alternatively, pySim-Shell can be used to view installed certificates**

```
pySIM-shell (00:MF)> select ADF.ARA-M
null
pySIM-shell (00:MF/ADF.ARA-M)>  aram_get_all
[
    {
        "response_all_ref_ar_do": [
            {
                "ref_ar_do": [
                    {
                        "ref_do": [
                            {
                                "aid_ref_do": "ffffffffffff"
                            },
                            {
                                "dev_app_id_ref_do": "e46872f28b350b7e1f140de535c2a8d5804f0be3"
                            }
                        ]
                    },
                    {
                        "ar_do": [
                            {
                                "apdu_ar_do": {
                                    "generic_access_rule": "always"
                                }
                            },
                            {
                                "perm_ar_do": {
                                    "permissions": "0000000000000001"
                                }
                            }
                        ]
                    }
                ]
            }
        ]
    }
]
```

#### Step 6: Install the Carrier Config Android app from Play Store

**Make sure the SIM card is placed in the default/first SIM slot of the device (only for multi-sim capable devices)**

Download the [CoIMS](https://play.google.com/store/apps/details?id=com.sherle.coims) Carrier Config app from play store. Then, run the app.

In the app, select **Force Enable IMS** to enable VoLTE/VoWiFi. The app will check for Carrier Privileges and if the app has Carrier Privileges, it will enable VoLTE/VoWiFi on the device.

#### Step 7: Additional IMS settings only for Samsung and Mediatek chipset devices

**Samsung IMS settings secret menu no longer works on devices running Android 10 and above. Use method mentioned here - https://xdaforums.com/t/tutorial-guide-how-to-access-ims-settings-on-modern-samsung-devices-without-computer.4646908/ instead.**

<p align="justify">
	After installation of the app, access the options menu on the right hand top corner and select Samsung/Mediatek IMS Settings option based on your device chipset and edit the IMS settings accordingly to enable desired IMS features
</p>

## OTA RAM (Remote Applet Management) + RFM (Remote File Management) of installing the applet and installing certificates

**This is an alternate method if the above method using GlobalPlatformPro tool does not work for you**

#### Step 1: Clone the sim-tools repository

```
$ git clone https://github.com/herlesupreeth/sim-tools

# Use shadysim.py if your SIM has ONLY USIM (not ISIM) application e.g. sysmoUSIM-SJS1-4ff
# Use shadysim_isim.py if your SIM has USIM + ISIM application e.g. sysmoISIM-SJA2
```

#### List all the applets installed on the SIM

```
# If SIM has ONLY USIM application
$ python shadysim.py --pcsc -t --kic <KIC1> --kid <KID1>

# If SIM has USIM + ISIM application
$ python shadysim_isim.py --pcsc -t --kic <KIC1> --kid <KID1>
```

#### Step 2: Copy the applet.cap file to sim-tools/shadysim folder and then install the applet

```
# If SIM has ONLY USIM application
$ python shadysim.py --pcsc -l applet.cap -i applet.cap  --kic <KIC1> --kid <KID1> --module-aid A00000015141434C00 --instance-aid A00000015141434C00

# If SIM has USIM + ISIM application
$ python shadysim_isim.py --pcsc -l applet.cap -i applet.cap  --kic <KIC1> --kid <KID1> --module-aid A00000015141434C00 --instance-aid A00000015141434C00
```

#### Step 3: Insert the following certificate into ARA-M applet on the SIM

The Carrier Config Android app is signed with following SHA1 key

SHA1: E4:68:72:F2:8B:35:0B:7E:1F:14:0D:E5:35:C2:A8:D5:80:4F:0B:E3

In order to provide Carrier Privileges to Carrier Config app, push the above SHA1 certifcate as follows

```
# If SIM has ONLY USIM application
$ python shadysim.py --pcsc --kic <KIC1> --kid <KID1> --aram-apdu 80E2900033F031E22FE11E4F06FFFFFFFFFFFFC114E46872F28B350B7E1F140DE535C2A8D5804F0BE3E30DD00101DB080000000000000001

# If SIM has USIM + ISIM application
$ python shadysim_isim.py --pcsc --kic <KIC1> --kid <KID1> --aram-apdu 80E2900033F031E22FE11E4F06FFFFFFFFFFFFC114E46872F28B350B7E1F140DE535C2A8D5804F0BE3E30DD00101DB080000000000000001
```

#### Step 4: List certificates loaded onto ARA-M applet

```
# If SIM has ONLY USIM application
$ python shadysim.py --pcsc --kic <KIC1> --kid <KID1> --aram-apdu 80CAFF4000

# If SIM has USIM + ISIM application
$ python shadysim_isim.py --pcsc --kic <KIC1> --kid <KID1> --aram-apdu 80CAFF4000
```

## Troubleshooting

For pySim and Sysmocom SIM cards related issues please use the following discourse - https://discourse.osmocom.org/c/sim-card-technology/

## Debugging
Use adb debugging with filter for "ims" keyword

## Note

### To test whether VoWiFi is enabled or not (assuming un-trusted 3gpp access via WLAN)

1. Program/Configure the PLMN of the SIM (and 4G network) to an operator who has wide deployment of VoWiFi (e.g. Airtel India (MCC: 404, MNC:45), Vodafone India, Reliance Jio India)

OR

1. Use a Samsung chipset based device, in which one can access the IMS settings. There, type in the ePDG DNS name or its IP address
2. Setup a DNS to resolve epdg.pub.epc.mncXXX.mccXXX.3gppnetwork.org and make sure UE gets the DNS IP address when it gets connected to WiFi AP
3. Have the setup required for VoWiFi ready (i.e. WiFi AP + ePDG + ePDG integrated with EPC) - More info about the architecture can be found [here](https://www.netmanias.com/en/post/oneshot/8127/lte-network-architecture-wi-fi-epdg/3gpp-based-lte-and-wi-fi-interworking-architecture-epdg-s2b)

## Potential reasons for this method not working
1. If the message in CoIMS show **Unrecognised Carrier Id. App may not work as intended** it means there is no CarrierId for that PLMN in AOSP - **Not shown in Android 8.0 and 8.1 devices**
	- If PLMN is on the following lists with respect to Android version the phone is running on

		https://android.googlesource.com/platform/packages/providers/TelephonyProvider/+/refs/heads/master/assets/latest_carrier_id/carrier_list.textpb

		https://android.googlesource.com/platform/packages/providers/TelephonyProvider/+/refs/heads/master/assets/sdk28_carrier_id/carrier_list.textpb

		https://android.googlesource.com/platform/packages/providers/TelephonyProvider/+/refs/heads/master/assets/sdk29_carrier_id/carrier_list.textpb

		https://android.googlesource.com/platform/packages/providers/TelephonyProvider/+/refs/heads/master/assets/sdk30_carrier_id/carrier_list.textpb

		Solution: Wait for vendor to release an update and hopefully it contains the updated carrier list
	- If PLMN is not on the above lists

		Solution: Refer the following link

		https://source.android.com/devices/tech/config/carrierid#integrating_carrier_ids_with_carrierconfig

2. If the SIM is placed in non-default SIM slot in a multi-SIM phones i.e. SIM in slot 1 (SIM slot 0 (default), SIM slot 1) of device
    - Solution: Place the SIM in default SIM slot (i.e. SIM slot 0) of the device
3. VoWiFi may not work on most devices because the ePDG address is still hard-coded in configuration files on the device and is not fetched from USIM
