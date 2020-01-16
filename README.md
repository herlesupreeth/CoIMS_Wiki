# CoIMS (Carrier Config overriding IMS settings)
Guide for overriding IMS settings to force enable VoLTE/VoWiFi using Carrier Privileges

## Requirements
- A programmable version of USIM/ISIM with KIC1, KID1 and KIK1, or a non-programmable USIM/ISIM with ARA-M application but with option to push certficates to ARA-M via OTA
- VoLTE/VoWiFi capable phone with Android Pie or above
- PCSC, serial card reader (SIM card programmer)
- Java v1.8

## My Setup
- sysmoUSIM-SJS1-4ff USIM with ADM keys
- OnePlus 5t UE with Android Pie
- Gemalto SIM programmer

## Big shout out and credits to following people for their awesome work
<p align="justify">
	@martinpaljak for GlobalPlatformPro (gp.jar) - A tool to load and manage applets on compatible JavaCards from command line
</p>

<p align="justify">
	@bertrandmartel for ARA-M applet (applet.cap) - ARA-M implementation for JavaCards. ARA-M is an application (typically present on a SIM card) which manage access rules that are enforced by an Access Control Enforcer (typically present on Android device). The enforcer makes sure the rules from the ARAM are enforced. An access rule is composed of an AID, a certificate hash (SHA1/SHA256 of client application cert) and a set of rules. The Access Control enforcer will allow/deny a client application (for example an Android app) to send APDU to a Secure Element (SE) applet based on these rules
</p>

## Steps

#### Install ARA-M Java Card applets on USIM/ISIM
<p align="justify">
	In order to install and/or manage Java Card applets on your card make sure you have KIC1, KID1 and KIK1 keys. KIC1, KID1 and KIK1 could differ from one SIM card to another so make sure to have the correct keys. If you have a non-programmable USIM/ISIM with ARA-M application and have option to push certficates to ARA-M via OTA, skip Step 1 and proceed to Step 2
</p>

Example: In sysmoUSIM-SJS1-4ff USIM cards, the key mappings for GlobalPlatformPro are as follows

sysmoUSIM key | GlobalPlatformPro argument
------------- | --------------------------
KIC1 | --key-enc
KID1 | --key-mac
KIK1 | --key-dek

**Proceed with caution when installing applets on SIM card as it could brick your USIM/ISIM if incorrect KIC1, KID1 and KIK1 keys are used**

```
$ git clone https://github.com/herlesupreeth/CoIMS_Wiki
$ cd CoIMS_Wiki
$ alias gp="java -jar $PWD/gp.jar"
```

<p align="justify">
	Please replace KIC1, KID1 and KIK1 with correct keys respective to your SIM card. The following command lists the details of the SIM card using the KIC1, KID1 and KIK1 keys and execution of it should not result in any error. If there is an error, please check the error and double check everything before proceeding
</p>

```
$ gp --key-enc KIC1 --key-mac KID1 --key-dek KIK1 -lvi
```

It is always better to unlock the SIM card for easier installation of applet as follows

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

Install the ARA-M applet (applet.cap) after unlocking. The following command must execute without any errors.

```
$ gp --install applet.cap
```

#### Push the SHA-1 certifcate of the Carrier Config Android app onto ARA-M in USIM/ISIM

The Carrier Config Android app which will be installed in next step is signed with following SHA1 key

SHA1: E4:68:72:F2:8B:35:0B:7E:1F:14:0D:E5:35:C2:A8:D5:80:4F:0B:E3

In order to provide Carrier Privileges to Carrier Config app, push the above SHA1 certifcate as follows

```
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

To check the list of installed certificates use the following command

```
$ gp -acr-list
RULE #0 :
       AID  : FFFFFFFFFFFF
       HASH : E46872F28B350B7E1F140DE535C2A8D5804F0BE3
       APDU rule   : ALWAYS(0x01)
```

<p align="justify">
	If you have a non-programmable USIM/ISIM with ARA-M application and have option to push certficates to ARA-M via OTA, push the above SHA1 certificate on to the SIM
</p>

#### Install the Carrier Config Android app from Play Store

Make sure the SIM card is placed in the default/first SIM slot of the device (only for multi-sim capable devices)

## Debugging
Use adb debugging with filter for "ims" keyword

## Potential reasons for this method not working
1. If the value of CarrierIdentifier indicated in the app is -1 (i.e Unknown Carrier)
	- If PLMN is on the following list (https://android.googlesource.com/platform/packages/providers/TelephonyProvider/+/master/assets/carrier_list.textpb)
		Resolution: Wait for vendor to release an update and hopefully it contains the updated carrier list
	- If PLMN is not on the following list (https://android.googlesource.com/platform/packages/providers/TelephonyProvider/+/master/assets/carrier_list.textpb)
		Resolution: Refer the following link (https://source.android.com/devices/tech/config/carrierid#integrating_carrier_ids_with_carrierconfig)

2. Some devices with Samsung Exynos chipset/ Mediatek chipset require ISIM, only USIM is not enough for SIP registrations

3. Does not seem to work on Samsung devices with Exynos chipset

4. If the SIM is placed in non-default SIM slot in a multi-SIM phones i.e. SIM in slot 1 (SIM slot 0 (default), SIM slot 1) of device


