Errol has generated a Windows CE 6.0 BSP by putting together some updates, fixes from the forum ([topic](http://www.friendlyarm.net/forum/topic/682)) and parameters that seem to be useful for the most of us. This BSP can be downloaded in the **Downloads** section: [MINI2440BSP.zip](http://friendlyarm.googlecode.com/files/MINI2440BSP.zip)

### Details ###
**Fixed:**
  * Network crash fix.
  * Multi CPU compile issue with SD driver. I hope. Couldn't test in Virtual Machine.

**Changed:**
  * Auto launch of touch screen calibration after first boot.
  * English language.
  * Network set to DHCP

**Removed:**
  * Chinese fonts

**Added:**
  * USB Remote NDIS Class driver (needed for Activesync)
  * Windows Console
  * SPI driver by domdom
  * Clock Synchronization registry entry, if SNTP client is included.
  * User Manager control panel applet by domdom, if FTP server or Telnet server or Webserver is included in the image.

**WARNING**: back up any old MINI2440 folders in PLATFORM or OSDesigns, as this will overwrite it.