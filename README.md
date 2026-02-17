# linux

Files :-https://github.com/VijayaIngale-shill/linux/blob/a21107d8cdb963ae952024d7afc9f39c411d9aa7/bin.docx

#! /bin/bash

help_function()
{
    echo "$0 provides information about files and folders including size and statistics"
    echo
    echo "$0 [-l location] [-e extension] [-s stats] [-h help]"
    echo
    echo "Examples:"
    echo "$0"
    echo "$0 -l /etc"
    echo "$0 -l /etc -e conf"
    echo "$0 -l /etc -e conf -s"
    exit 2
}

# Default values
LOCATION=$(pwd)
EXTENSION=""
STATS=0

# Read arguments
while [ $# != 0 ]
do
    case $1 in
        -l | --location)
            LOCATION=$2
            if [ ! -d "$LOCATION" ]; then
                echo "Invalid directory"
                exit 1
            fi
            shift 2
            ;;
        -e | --extension)
            EXTENSION=$2
            shift 2
            ;;
        -s | --stats)
            STATS=1
            shift
            ;;
        -h | --help)
            help_function
            ;;
        *)
            echo "Invalid option"
            help_function
            ;;
    esac
done

echo "Analyzing location: $LOCATION"
echo

# Count folders
FOLDER_COUNT=$(ls -l "$LOCATION" | awk '/^d/' | wc -l)

# Count files
if [ "$EXTENSION" = "" ]; then
    FILE_LIST=$(ls -l "$LOCATION" | awk '/^-/')
else
    FILE_LIST=$(ls -l "$LOCATION" | awk '/^-/' | grep ".$EXTENSION$")
fi

FILE_COUNT=$(echo "$FILE_LIST" | wc -l)

# Calculate total size
TOTAL_SIZE=$(echo "$FILE_LIST" | awk '{sum += $5} END {print sum}')

# Display results
echo "Number of folders: $FOLDER_COUNT"
echo "Number of files: $FILE_COUNT"
echo "Total file size: $TOTAL_SIZE Bytes"
echo "Total file size: $(echo "$TOTAL_SIZE/1024" | bc) KB"
echo "Total file size: $(echo "$TOTAL_SIZE/1024/1024" | bc) MB"

echo

# Statistics if enabled
if [ $STATS -eq 1 ] && [ "$FILE_COUNT" -gt 0 ]; then

    echo "$FILE_LIST" | awk '
    NR==1 {
        min=$5
        max=$5
        min_file=$9
        max_file=$9
    }
    {
        if ($5 < min) {
            min=$5
            min_file=$9
        }
        if ($5 > max) {
            max=$5
            max_file=$9
        }
    }
    END {
        print "Statistics:"
        print "Smallest file: " min_file " (" min " Bytes)"
        print "Largest file: " max_file " (" max " Bytes)"
    }'

fi
