# The Problem
When attempting to execute a MyST action you receive the following error:
![](/platform-configuration/javalangnullpointerexception/npe_01.png)

# The Cause
A MyST property is set to an empty string resulting in a null or None object. Usually occurs when you define a property's value and later clear the value.
![](/platform-configuration/javalangnullpointerexception/npe_02.png)

1. For example the screenshot below shows the __Custom Identity Keystore__ defined.
![](/platform-configuration/javalangnullpointerexception/npe_03.png)
2. Later you may wish to unset the __Custom Identity Keystore__ by deleting the property's value.
![](/platform-configuration/javalangnullpointerexception/npe_04.png)
3. This results in the property being a null or None type object
![](/platform-configuration/javalangnullpointerexception/npe_05.png)

# The Solution
Reset properties that are not used. Reset can achieve the following:
* restore the property to its auto-computed value (eg. resetting AdminServer listen port will reset to 7001)
* restore the property to its inherited blueprint value (if resetting from Platform Model)
* removes the property (if not an auto-computed property)

To reset a property:

1. When in edit you can highlight over the property name and click the '__X__' button to reset a property.
![](/platform-configuration/javalangnullpointerexception/npe_06.png)
2. Now the __Custom Identity Keystore__ is no longer displayed.
![](/platform-configuration/javalangnullpointerexception/npe_07.png)
3. Remember to Save and Commit your changes.
