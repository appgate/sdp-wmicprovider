# Deprecation warning
Microsoft has deprecated wmic in Windows 10 and removed wmic in Windows 11. 
The sdp-wmicprovider is as is and will be deprecated whith the same terms.

# Overview
We encourage you are familiar with the [Appgate Extension](https://github.com/appgate/sdp-extensions/) before continuing.

## The WMI
The Windows Management Instrumentation Command-line (WMIC) utility provides a command line interface. This can be very handy when you need to elaborate information gathering on Windows devices when establishing policy enforcement with the help of criteria scripts and conditions. For example, you can determine if the machine contains any attached USB flashcards and act on the fact (e.g., allow or block).

* [WMIC reference on MSDN](https://msdn.microsoft.com/en-us/library/aa394531%28v=vs.85%29.aspx)

WMIC provides a large set of direct calls into the WMI and presents the results in a tabular format. While the information might be useful, it is not directly presented in a usable (parse-able) format for scripting. We provide a wrapper that reformats the results into an "array of JSON strings"--hence, it can be easily parse-able. This brings another benefit: You don't need to write your own script--you can use the WMIC wrapper provided.

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
This condition can then be attached to a entitlement to enforce this type of compliance check.
 

