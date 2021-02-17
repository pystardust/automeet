# Automeet
Automatically join online classes.

Many of us need to attend classes online, and finding the right url for each class and clicking on it at the appropriate time is such a pain (I am too lazy to do all that).

So here is a solution I found out to make my life a little easier.
I started using **calcurse** to maintain my class time table. It is a beautiful tui application which manages events and todos. It has this amazing feature to run a command just before an event start. The default command would send a notification regarding the upcoming event. But this command can be changed to anything. Changing this command to open the class url would automatically open the class for you.

# Requirements
* calcurse

# Procedure

#### Adding Timetable

* Added each class as a recurring event in calcurse with the title having the class name and the class url as follows
```
Topology <Tab Space> https://meet.google.com/url123
```

#### Setting up script to open url
* Now when I run calcurse --next from the command line I get output as follows
```
$ calcurse --next                                                
next appointment:
   [08:26] Topology		https://meet.google.com/url123
```
> This means the next class is topology and it is in 8hr 23min and the joining link is that.

* Here is a small script that would notify about the next event and also open the url if it exists.
```
#!/bin/sh

browser="brave"
event="$(calcurse --next | sed "1d")"
name="$(echo "$event" |  sed -E 's/.*\] ([^\t]*).*/\1/')"
url="$(echo "$event" | grep -o "http\S*" )"

if [ -z "$url" ] 
then
	notify-send "$name"
else
	notify-send "$name" "opening link..." 
	eval $browser $url 1>/dev/null 2>&1 & 
fi
```
Save this as an executable somewhere

#### Configuring Calcurse to run the script
Now all that is left to do is to configure calcurse to run this script 15 seconds before the event starts. 
* Add the following lines to calcurse config file (~/.cache/calcurse/conf).
```
notification.command= /home/user/path/to/this/script
notification.notifyall=all
notification.warning=15
```
And there we have it. Now whenever calcurse is open (or launched as daemon), all your classes will be automatically opened in the browser.

## Extra

* If you are really adventurous there are browser plugins that automatically mute, switch off camera and auto join you into google meet.
* I also add the following as a module in my status bar script to show when I have my next class.
```
event="$(calcurse -n | sed 1d | sed -E "s_^ *\[(.*):(.*)\] ([^\t]*)\t?.*_[\1h \2m->\3]_")" 
```
This would give an output as *07h 27m -> Topology*.

