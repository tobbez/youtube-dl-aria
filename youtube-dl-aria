#!/bin/sh

UA=`youtube-dl --dump-user-agent`
TMPDIR=`mktemp -d`
COOKIES="$TMPDIR/cookies"

trap "rm -rf $TMPDIR" 0

ARIA_DNS_FLAGS=""
aria2c -h#all|grep -- '--async-dns' >/dev/null 2>&1
if [ $? -eq 0 ]
then
	ARIA_DNS_FLAGS="--async-dns=false"
fi

# We need to separate the execution of youtube-dl and invocations of
# aria2, since youtube-dl writes cookie data after it has performed
# /all/ of its work.
youtube-dl -o "[%(upload_date)s][%(id)s] %(title)s (by %(uploader)s).%(ext)s" --get-url --get-filename --cookies=${COOKIES} "$@" > ${TMPDIR}/video_data

while read URL && read FILENAME
do
	CLEANED_FILENAME=`echo "${FILENAME}" | tail -n 1 | tr ":\"" ";'" | tr -d "\\\/*?<>|"`

	echo "$CLEANED_FILENAME"
	aria2c $ARIA_DNS_FLAGS -c -j 3 -x 3 -s 3 -k 1M --load-cookies="$COOKIES" -U "$UA" -o "$CLEANED_FILENAME" "$URL"
done < $TMPDIR/video_data
