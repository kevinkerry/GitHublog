title: Lock Mechanism Of Mac
date: 2014-08-08 16:34:01
tags: [Security, Mac, Objective-C]

---

## System Authentication Service & Lock of Mac OS

<!--more-->

## Basic knowledge

The lock on iTIS is kind of “system authentication service”. There are some different concepts for this service:

1. **Right**: a named privilege that the application requests on behalf of a user.
2. **Credential**:  a token representing an authenticated user that the Security Server stores as part of the authorization session.
3. **Rule**: a set of attributes that determine who should be authorized to perform a specific action.

When doing authorization, the application sends a Right request to OS X, waiting for the system to match particular Rule to this Right.

If the corresponding Credential doesn’t exist in the database, the system will ask user to do the authenticate according to the attributes in the Rule.

The default Rule is to ask the password and username of an administrator. Once the system gets the pwd and name, it will grant the authorization and create the Credential in database for next request. The default timeout of this Credential is 300 seconds.
 
## Back to iTIS

The Right that iTIS request to system uses the default Rule – asking for administrator’s credential – which would be shared by several system settings and probably also shared with other different apps.

But even for the System Preferences, different settings have different credential. That’s why you will find changing the status of the lock icons for some items can impact iTIS while the others can’t. Currently, we test that the following (but may not only these):

* items needs administrator credential: Energy Saving, Startup disk, Sharing …  Changing these item’s lock status would impact iTIS.

* Some other items like Account, Security … would not impact iTIS as they are requiring even higher rights (root privilege, not administrator)
 
## What’s more

As iTIS requests for the administrator credential, the current login user group would also impact iTIS.

* If you are logging as an administrator account, iTIS would be unlocked from beginning of the system startup or account login, because at this time, the system already has the credential while you typed in the name and password when login the OS. (Actually, at this scenario, the preference items that require administrator privilege described above even don’t display the lock icon at all).

* Otherwise, if you are using a non-administrator account, such as a guest account or a standard account, iTIS would be locked form the beginning of startup. That’s why you have different status reports at different times.
 
To recap, the lock status depends on the whole system status, not only the explicit updates of iTIS itself. The result is a complicated combination of several different parts of the system and other 3rd party Apps.