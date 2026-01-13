# Optional Startup Definition File (fabmo-def.json)

For the initial boot of the FabMo software (no files existing yet in: /opt/fabmo), the definition file in this folder provides a designated profile to boot to.

When the FabMo software is first started, it will load the default profile information; then it will automatically restart and boot to the profile designated here.

This double-start boot-up process only happens once. If this file is missing or is set to the default profile, then the boot will only be to the default settings.

As of 9/1/25 the following profiles are available (these are the full profile names for use here, not the display names):

-  fabmo-profile-dt
-  fabmo-profile-dtmax
-  fabmo-profile-dtatc
-  fabmo-profile-handibot-2
-  default

*You may need to reboot a second time for the full IP address signalling system to be established.