## Disclaimer
© 2019, AppGate, Inc.  All rights reserved. 
Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met: (a) redistributions of source code must retain the above copyright notice, this list of conditions and the disclaimer below, and (b) redistributions in binary form must reproduce the above copyright notice this list of conditions and the disclaimer below in the documentation and/or other materials provided with the distribution.
THE CODE AND SCRIPTS POSTED ON THIS WEBSITE ARE PROVIDED ON AN “AS IS” BASIS AND YOUR USE OF SUCH CODE AND/OR SCRIPTS IS AT YOUR OWN RISK.  APPGATE DISCLAIMS ALL EXPRESS AND IMPLIED WARRANTIES, EITHER IN FACT OR BY OPERATION OF LAW, STATUTORY OR OTHERWISE, INCLUDING, BUT NOT LIMITED TO, ALL WARRANTIES OF MERCHANTABILITY, TITLE, FITNESS FOR A PARTICULAR PURPOSE, NON-INFRINGEMENT, ACCURACY, COMPLETENESS, COMPATABILITY OF SOFTWARE OR EQUIPMENT OR ANY RESULTS TO BE ACHIEVED THEREFROM.  APPGATE DOES NOT WARRANT THAT SUCH CODE AND/OR SCRIPTS ARE OR WILL BE ERROR-FREE.  IN NO EVENT SHALL APPGATE BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, RELIANCE, EXEMPLARY, PUNITIVE OR CONSEQUENTIAL DAMAGES, OR ANY LOSS OF GOODWILL, LOSS OF ANTICIPATED SAVINGS, COST OF PURCHASING REPLACEMENT SERVICES, LOSS OF PROFITS, REVENUE, DATA OR DATA USE, ARISING IN ANY WAY OUT OF THE USE AND/OR REDISTRIBUTION OF SUCH CODE AND/OR SCRIPTS, REGARDLESS OF THE LEGAL THEORY UNDER WHICH SUCH LIABILITY IS ASSERTED AND REGARDLESS OF WHETHER APPGATE HAS BEEN ADVISED OF THE POSSIBILITY OF SUCH LIABILITY.   



# Overview
We encourage you are familiar with the [AppGate Extension](https://appgate.github.io/appgate-extensions/) before continuing.

## The WMI
The Windows Management Instrumentation Command-line (WMIC) utility provides a command line interface. This can be very handy when you need to elaborate information gathering on Windows devices when establishing policy enforcement with the help of criteria scripts and conditions. For example, you can determine if the machine contains any attached USB flashcards and act on the fact (e.g., allow or block).

* [WMIC reference on MSDN](https://msdn.microsoft.com/en-us/library/aa394531%28v=vs.85%29.aspx)
* [Examples of useful WMIC queries](https://blogs.technet.microsoft.com/askperf/2012/02/17/useful-wmic-queries/)

WMIC provides a large set of direct calls into the WMI and presents the results in a tabular format. While the information might be useful, it is not directly presented in a usable (parsable) format for scripting. We provide a wrapper that reformats the results into an "array of JSON strings"--hence, it can be easily parsable. This brings another benefit: You don't need to write your own script--you can use the WMIC wrapper provided.

## Example
Use-case: deny any access in an entitlement (or policy), if the user has a USB drive attached to the machine the client is running on. The following example works on Windows 10 down to latest Windows 7; however, we do not maintain compatibility for it (no 32 bit).

### Get wmicprovider.exe

* [latest release](https://github.com/appgate/appgate-wmicprovider/releases/latest)

Upload the wmicprovider.exe to the on-demand device scripts to your controller. The on-demand device claim mapping on the identity provider looks something like this:

* script: wmic provider
* args: diskdrive get InterfaceType,caption,Mediatype,size
* Claim Name: diskdrive
* Platforms: desktop.windows.all
* An example of running the above command:

```json
[ 
    { 
        "Caption": "LITEONIT LMT-128M6M mSAT", 
        "InterfaceType": "IDE", 
        "MediaType": "Fixed hard disk media", 
        "Size": 128034708480 
    }, 
    { 
        "Caption": "Generic Flash Disk USB Device", 
        "InterfaceType": "USB", 
        "MediaType": "Removable Media", 
        "Size": 1998743040 
    } 
]
```
A condition can now be crafted with the value, stored in `claims.device.diskdrive`. For the condition, you could create the following javascript expression:
```js
try{
    var jobs = JSON.parse(claims.device.diskdrive);
    for (var i = 0; i < jobs.length; i++){
            var obj = jobs[i];
            if (obj.InterfaceType == "USB") {
                   return false;
            }
    }    
}
catch(err){
     console.log(err);
     return false;
}
return true;

```
This condition can then be attached to a entitlement to enforce this type of compliancy check.
 

