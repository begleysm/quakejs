#!/bin/sh
# Merges the .pk3-files in ./myAssets with a file of your choice
# Adds the crc32-value in front of the pak3-file and creates the info you
# need to add to manifest.json
#
# I you add maps do not forget to update your mapcycle in 
# base/baseq3/server.cfg! 
#
# You need to have libarchive-zip-perl installed (calculating the  crc32 checksum)
# sudo apt install libarchive-zip-perl
#
# You need to have zipmerge installed (merging the files)
# sudo apt install zipmerge
##
# Maybe you have to install bash as well (if you use Ubuntu it should already
# be installed)
# sudo apt install bash
#
# The easiest way is to merge your files with either pak100.pk3 or pak101.pk3 
# since you do not need to mess with the nodescript files and both files
# offer a lot of space for your maps and models. There seems to be a file-size-restiction 
# when loading pak-files. Try to keep them under 75 MB!
#
#Example: ./mergescript ../base/baseq3/pak100.pk3

file="$(basename $1)"
tempName="${file}-temp"

echo "Attempting to copy $file in the current directory..."
cp $1 ./$tempName
echo "Done"
echo ""
echo "Attempting to merge files..."


#Delete everything not .pk3 from the myAssets-folder
if [ -n "$(ls -I "*pk3" ./myAssets)" ]
then
rm -f ./myAssets/$(ls -I "*pk3" ./myAssets)
fi

zipmerge $tempName ./myAssets/*
echo "Done"
echo ""
echo "Calculating CRC32 checksum..." 
checksum=$((0x$(crc32 $tempName)))
echo "" 
newName="$checksum-$file"
echo "Renaming $tempName to $newName"
mv $tempName $newName
echo "Done"
echo ""
echo "Now copy the new file in /var/www/html/assets/baseq3 and in ../base/assets/baseq3"
echo "You can copy the files with the following commands (make backup of original files first!)"
echo ""
echo "sudo cp $newName /var/www/html/assets/baseq3"
echo "sudo cp $newName ../base/baseq3/$file"
echo ""
filesize="$(stat -c%s $newName)"

if [ $filesize -gt 75000000 ]; then
echo ""
echo "***Your file is close to / or larger than 75MB. This can prevent the file from being loaded by your browser...***"
fi
echo ""
echo "Please update your /var/www/html/assets/manifest.json with this:"
echo ""
echo "  {"
echo "     \"name\": \"baseq3/$file\","
echo "     \"compressed\": $filesize,"
echo "     \"checksum\": $checksum"
echo "  },"
echo ""
echo "Open the manifest with nano and make the changes:" 
echo "sudo nano /var/www/html/assets/manifest.json"
echo ""
echo "STRG+o to save changes"
echo "STRG+x to exit nano"
echo ""
echo "If you have added maps you want to adjust your mapcycle in server.cfg"
echo "sudo nano ../base/baseq3/server.cfg"
echo ""
echo "Restart Apache webserver with"
echo ""
echo "sudo service apache2 restart"
echo ""
echo "and (re)start you quakejs server"
