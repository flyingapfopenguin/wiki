---
title: "Radio Script"
date: 2023-04-06T22:54:01+02:00
---

We use the following scipt and list of radio stations to stream radio from the command line.

Note: The Bash script uses vlc / mplayer.

Script:

```bash
#!/bin/bash
. <path>/radio_station_list # TODO

if [ $# -eq 0 ]
then
        readarray -t sorted < <(for a in "${!stations[@]}"; do echo "$a"; done | sort)
        echo "Configured Stations:";
        echo "====================";
        for stn in "${sorted[@]}"; do 
                stationName_length=`expr length "$stn"`;
                target_length=`expr 20 - $stationName_length`;
                blankString=" ";
                while [ `expr length "$blankString"` -lt $target_length ]; do
                        blankString="$blankString ";
                done
                echo "   $stn $blankString - ${stations["$stn"]}"; 
        done
        echo ""
        echo "`basename $0` {station}       -> plays the radio station"

elif [ $# -eq 1 ]
then 
  stn=$1;
	url=${stations["$stn"]};
	url_ext=`echo $url | tail -c 5`
	echo $url_ext;
	if [ "$url_ext" == ".m3u" ]
	then
		cvlc $url;
        elif [ "$url_ext" == ".mp3" ]
        then
                cvlc $url;
        elif [ "$url_ext" == ".pls" ]
        then
                cvlc $url;
	else
        	mpg123 $url;
	fi
else
  echo "Benutzung: `basename $0`                -> prints a list of configured radio stations";
  echo "Benutzung: `basename $0` {station}      -> plays the radio station";
  exit -1
fi
```

radio\_station\_list:

```bash
#!/bin/bash
declare -A stations=(
['dlf']='http://www.deutschlandradio.de/streaming/dlf.m3u'
['ndr1']='http://www.ndr.de/resources/metadaten/audio/m3u/ndr1niedersachsen_ol.m3u'
['antenneAC']='http://mp3.antenneac.c.nmdn.net/ps-antenneac/livestream.mp3'
['awesome80s']='http://www.181.fm/winamp.pls?station=181-awesome80s&file=181-awesome80s.pls'
['lite90s']='http://www.181.fm/winamp.pls?station=181-lite90s&file=181-lite90s.pls'
)
```