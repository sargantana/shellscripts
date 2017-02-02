#!/bin/bash
#Colors
R="\e[1;31m" # red: Warning message
Y="\e[1;33m" # yellow: script Error
G="\e[1;32m" # green: Standard message
C="\e[1;33m" # CYAN: Questions
D="\e[0;m" # default system console color: Normal :: make last in colors

HTTPS=false
code="/tmp/code.txt"
player=mplayer
key=false
audio=false 

#Functions
dohelp () {
cat << EOF
    Usage: ${0##*/} [-kahs] URL|KEY
    Download youtube videos or audio using a URL or Video ID 

       -k          Use a Video ID
       -a      Download an audio file
       -s      Force https
       -h          This help
EOF
    section "Examples"
    echo -e "Usage with url to get video: ${0##*/} http://www.youtube.com/watch?v=OuSdU8tbcHY"
    echo -e "Usage with key to get video: ${0##*/} -k OuSdU8tbcHY "
    echo -e "Usage with key to get audio: ${0##*/} -k -a OuSdU8tbcHY"
    echo -e "Usage with url to get audio: ${0##*/} -a http://www.youtube.com/watch?v=OuSdU8tbcHY$"
    echo -e "This help: ${0##*/} -h"
}

promptyn () {
    while true; do
        echo -e "${C}$1[Yes]?${D}"
        read -e -p "Yes/no  " yn
        if [ -z "$yn" ];then
           yn=Y
        fi
        case $yn in
            [Yy]* ) return 0;;
                [Nn]* ) return 1;;
        esac
    done
}

section () {
    echo "________________________________________________"
    echo ""
    echo -e "${G}$1${D}"
    echo "________________________________________________"
    echo ""
}

itag () {
    v=$(grep -i "itag%3D${1}" $code)
    if [ -z "$v" ];then
        echo -e "Not Found Itag:${Y}$1${D} Quality:${Y}$2${D}"          
    else
        echo -e "Found: Itag:${Y}$1${D} Quality:${Y}$2${D}"
        l=$(percentdecode.perl "$v")
        if $HTTPS;then
            link=$(sed 's/http\:/https\:/g' <<< "$l")
        else
            link="$l"
        fi
        echo -e "Downloading ${G}$name ${Y}$2${D}" && wget --no-check-certificate -O "$name" "$link" && section "Done"
        rm -f $code
        if promptyn "Run with $player "; then
            $player "$name"
        fi
        exit 0
    fi
}

readopts () {
  while getopts "akshp:" optname
    do
      case "$optname" in
        "a")
      audio=true    
          ;;
        "k")
      key=true
          ;;
    "s")
          HTTPS=true
          ;;
    "p")
          player=$OPTARG
          ;;
    "h")
          section "Help"
      dohelp
      exit 0
          ;;
        ":")
          echo "No argument value for option $OPTARG"
          ;;
        *)
        # Should not occur
      section "Help"
      dohelp
      exit 1
          ;;
      esac
    done
}

readargs () {
  for p in "$@"
    do
      url="$p"
    done
}

section "Youtube downloader"
readopts "$@"
argstart="$?"
readargs "${@:$argstart}"

if [ -z "$url" ];then
    echo -e "Error: ${R}Invalid URL${D}"
    dohelp
    exit 1
else
    if [ -z "${url:3:4}" ];then
        echo -e "Error: ${R}Invalid URL ${url} ${D}"
        dohelp
        exit 1
    fi
    if ! $key;then
        if $HTTPS;then
            echo -e "Forcing https"
            u=$(sed 's/http\:/https\:/g' <<< "$url")
        else
            u="$url"
        fi
        if [ "${u:0:4}" != "http" ];then
            echo -e "Error: ${R}Invalid URL ${u} ${D}"
            dohelp
            exit 1
        fi

    else
          echo "Using key"
      u="https://www.youtube.com/watch?v=$url"

    fi
    echo -e "Found URL ${Y} $u ${D}"
    name=$( ( cut -d '=' -f 2 <<< "$u" ) | cut -d '&' -f 1).mp4

fi

echo -e "Downloading the code"

wget --no-check-certificate -q -O - "$u" > "$code"
echo -e "Done"
echo -e "Parsing"
( tr '"' '\n' < $code ) > ${code}.tr
sed -i -r 's/\\u0026/\n/g' ${code}.tr
sed -i -r 's/,/\n/g' ${code}.tr
grep -i "url" ${code}.tr | grep -i "googlevideo.com" > "$code"
sed -i -r 's/url=//g' "$code"
rm -f ${code}.tr


if $audio;then
        echo "Getting Audio"
    itag 141 246kbps
    itag 172 192kbps
    itag 140 128kbps
    itag 171 128kbps
    itag 139 48kbps
else
        echo "Getting Video with sound"
    itag 37 1080p
    itag 46 1080p
    itag 45 720p
    itag 22 720p
    itag 35 480p
    itag 44 480p
    itag 34 360p
    itag 43 360p
    itag 18 360p
    itag 5  240p
    itag 36 240p
    itag 17 144p
fi

rm -f "$code"
section "Link not found"
