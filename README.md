# hass
A home for some miscellaneous HomeAssistant Stuff....

Some short info per file below:

<br>

#### update_hass 
****
A small script to automate the update of Home Assistant in docker.

<br>

* Script requires root priviliges, run as root or sudo!
* Script has no function for user input, all changes need to be done in script file prior to first run!
* Requires the container name to be 'Home Assistant' and that docker is running on local machine.(This isn't a varible so change on own risk!)
* Default config path for the container is /home/pi/hass , change if ypur path is different.
* If you use RFX-com for all your 433MHz needs, the path is also assumed, check and change if necessary!
* Container is set to restart on container failure with a maximum of five(5) retries, change if you want a different behavior, explanation is in script file.
* Doesn't remove old images, this still needs to be done manually.
