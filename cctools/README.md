# Adding Maps & Custom Content in QuakeJS

Written by [digidigital](https://github.com/digidigital/)

The repak.js-script that rearrages map-files so they can be added to the game seems to no longer work with current versions of node.

### Option 1: Add your files to pak100.pk3 and pak101.pk3
If you don´t want to optimize/customize your pak0 (remove maps / models to save drive space, adjust bot files, etc.) there is a much simpler way. All your maps, models and textures can simply go into pak100.pk3 and pak101.pk3. Those files are small and there is a lot of space for your files left. You simply put the merged files in the right places and update one json-file and your server.cfg. I have written a script that copies your custom .pk3 content into Quakejs´s .pk3s and calculates the correct checksums and manifest.json info for you. So it's really easy ;)  

### Option 2: Customize pak0.pk3 
You can add custom content like textures, models, maps to the pak0.pk3 of the Q3A-Demo that is used to obtain the proprietary art assets. [Grabisoft](https://github.com/grabisoft) has made a great [video](https://www.youtube.com/watch?v=m57rMXASWms&feature=youtu.be) that I used as the foundation for this guide regarding the (more complex) pak0.pk3 modification

**As a prerequisite I assume you have installed a "vanilla" server of quakejs running as user "quake" as described in this
[tutorial](https://steamforge.net/wiki/index.php/How_to_setup_a_local_QuakeJS_server_under_Debian_9_or_Debian_10#The_Simpler_Method).
It is important that you have checked your server is running fine in the default configuration.
The server should be stopped while you make all the changes.**
 
# Adding maps, models and textures the easy way

The games content is stored in .pk3-files (basically zip-files). There seems to be a file-size-restiction in the browser when loading pk3-files. Try to keep them under 75 MB!

Since this is the "easy" option I don't go into details of what we are doing and provide you with a simple step-by-step guide. If you need more info or run into problems just read the more complex guide down below ;) 

We will use the **mergescript** to merge your files with the already existing ones.

#### mergescript
The scripts need libarchive-zip-perl to calculate the checksum and zipmerge for merging the pk3-files
```shell
sudo apt install libarchive-zip-perl zipmerge
```

In case the script ist not executable 
```shell
sudo chmod +x mergescript 
```
Put all the **.pk3**-files (maps, models, textures) in the ./myAssets folder. *Do not forget to **unzip your downloads!** Make sure you do not put a zip-file there!* The mergescript will delete everything in the myAssets-folder without .pk3-extension before it merges the files.

Now execute the mergescript and point it to the file you would like to be the "target" of the merge-operation. The script will copy it into the current folder and add your files. Good candidates are pak100.pk3 and pak101.pk3 that have been downloaded to ~/quakejs/base/baseq3 during the installation. 

Type
```shell
./mergescript ../base/baseq3/pak100.pk3
```
or
```shell
./mergescript ../base/baseq3/pak101.pk3
```
and you should get something like this
```shell
User@Ubuntu-PC:~/quakejs/cctools$ ./mergescript ../base/baseq3/pak100.pk3
Attempting to copy pak100.pk3 in the current directory...
Done

Attempting to merge files...
Done

Calculating CRC32 checksum...

Renaming pak100.pk3 to 2586809043-pak100.pk3
Done

Now copy the new file in /var/www/html/assets/baseq3 and in ../base/assets/baseq3
You can copy the files with the following commands (make backup of original files first!)

sudo mv 2586809043-pak100.pk3 /var/www/html/assets/baseq3
mv 2586809043-pak100.pk3 ../base/baseq3/pak100.pk3

Please update your /var/www/html/assets/manifest.json with this:

  {
     "name": "baseq3/pak0.pk3",
     "compressed": 18856192,
     "checksum": 2586809043
  },

Open the manifest with nano and make the changes: 
sudo nano /var/www/html/assets/manifest.json

STRG+o to save changes
STRG+x to exit nano

If you have added maps you want to adjust your mapcycle in server.cfg
sudo nano ../base/baseq3/server.cfg

Restart Apache webserver with

sudo service apache2 restart

and (re)start you quakejs server
```
If you follow all the steps in mergescript's output and your pk3 is below 75MB you should have some new stuff on your server!

...from time to time you should delete the old pak100-files from the /var/www/html/assets/baseq3-folder.

# The more complicated way to add content by modifying pak0.pk3

## PK3 Files
In order to change a pk3 file you simply rename it to .zip and open it with you preffered ZIP-manager software. After you have copied, deleted or added files you simply close the file an rename the extension back to .pk3 

First
```shell
mv pak0.pk3 pak0.zip
```
Add or delete content with a ZIP-Manager then
```shell
mv pak0.zip pak0.pk3
```

## Adding content to pak0

If you want to add a model, skin or map just copy the files from the pk3 into the same folders in the pak0.pk3 as described above (Rename file extension to .zip -> copy files/folders -> paste in your .pk3 -> close files -> rename extension back to .pk3)

In order to keep the size low you might want to delete the original maps from the maps folder (.bsp & .aas) and replace the videos in the video folder with empty files with the same name.

## Preparing your server

In order for your server to accept the changes in pak0.pk3 you need to make some changes in some text files and copy your pak to the right places.

### .js-files

You need to remove the reference to pak0.pk3 in the following files 

```shell
sudo nano ~/quakejs/build/ioq3ded.js
```

```shell
sudo nano ~/quakejs/build/ioquake3.js
```

```shell
sudo nano /var/www/html/ioquake.js
```

Press "CTRL + w" and search for "linuxq3ademo"

Delete **exactly** this from the files:

*{name:"linuxq3ademo-1.11-6.x86.gz.sh",offset:5468,paks:[{src:"demoq3/pak0.pk3",dest:"baseq3/pak0.pk3",checksum:2483777038}]},*

Press "CTRL+o" to write the file and "CTRL+x" to exit nano.

### Copy pak0.pk3

Delete the old pak0 from the assets folder
```shell
sudo rm /var/www/html/assets/baseq3/*-pak0.pk3
```

Copy your pak0.pk3 in these folders
```shell
sudo cp /path/to/your/file/pak0.pk3 ~/quakejs/base/baseq3
sudo cp /path/to/your/file/pak0.pk3 /var/www/html/assets/baseq3
```

The file in the *web-server-folder* (**not the one in your home folder!!!**) needs to be renamed. You need the files CRC32 checksum as a prefix. This checksum needs to be added to the manifest.json in a later step as well.

**The result should look like this:**
12345678-pak0.pk3

### Creating the checksums

I have made two scripts that help you to calculate the checksum for pak-files (you need to know it in order to rename the files in your /var/www/html/assets/baseq3/ folder and to adjust the values in manifest.json)

*If are not sure which tool/shell command to execute take "crc-rename" -> Example: "Rename a single file" **after** you have copied pak0.pk3 in the webserver folder.* 

Both scripts need libarchive-zip-perl to calculate the checksum

```shell
sudo apt install libarchive-zip-perl
```
#### crc32info
Use this to get a list of CRC32 / filename values

In case the script ist not executable 
```shell
sudo chmod +x crc32info 
```

**Examples**

Get the crc32-info for one file
```shell 
 ./crc32info /var/www/html/assets/baseq3/pak0.pk3
```

Calculate the crc32-info for all pk3-files in a directory
```
./crc32info /var/www/html/assets/baseq3/*pk3
```
#### crc32-rename
Use this to batch rename pk3-files. I just added sudo in front of the command in case you are not root ;)

In case the script ist not executable 

```shell
sudo chmod +x crc-rename
```

In case you do not have bash installed...
``` shell
sudo apt install bash
```

**Examples**

Rename a single file 
```
sudo ./crc-rename /var/www/html/assets/baseq3/pak0.pk3
```
Rename more than one file 
```
sudo ./crc-rename filename1 filename2 ...
```

Rename all pk3's in other directories
```
sudo ./crc-rename /var/www/html/assets/base*/*
```

### manifest.json

Open manifest.json and add yor pak0.pk3's information

```shell
sudo nano /var/www/html/asssets/manifest.json
```

**DELETE** this part
```
{
    "name": "linuxq3ademo-1.11-6.x86.gz.sh",
    "compressed": 49292197,
    "checksum": 857908472
  },
```

After the info for pak101.pk3 **ADD** (**replace file size and checksum with the values for your file!!!**)
"Compressed" is the size in bytes. It seems it doesn't really matter if this value is not correct. Please note that the filename in the manifest does **not** contain the checksum.

```
  {
    "name": "baseq3/pak0.pk3",
    "compressed": 74500000,
    "checksum": 1638991753
  },
```

Press "CTRL+o" to write the file and "CTRL+x" to exit nano.

## Final steps
* Update mapcyle in server.cfg with new maps
* Restart your webserver
```shell
sudo service apache2 restart
```
* Start your quakejs-Server

## Common issues / fixes
If the server starts fine but you can't connect or the games does not load you should check if you have followed all the steps from this tutorial. 

Things I ran into:
* My pak0.pk3 was too large (>~75MB)
* I did not restart the webserver and for some reason this caused issues
* I did not restart the quakejs server :)
* My browser did not have the latest content (clear browser cache and saved data)
* Typos while renaming files or adjusting the manifest.json
* Forgot to adjust the values in manifest.json
* I deleted files from pak0.pk3 that were needed to start the game
* Forgot to put the updated pak0.pk3 in both folders 
 
# How to find what is missing...

## Identify missing textures
When you put custom maps in the pak0.pk3 you may run into the issue that you have a lot of missing textures.

This issue can be solved by manually adding the missing textures. If you use the demo pak0.pk3 you can take the CC-licensed hi-res textures from [here](https://ioquake3.org/extras/replacement_content/) or in case there is still something missing from openarena. The openarena-zip contains a pak4-textures.pk3

In order to find out what is missing start the map in your browser and open the console by pressing

~ or ^

in quakejs.

Check for any lines in bold and yellow with a "WARNING". If it says a .tga or .jpg is missing you have found a missing texture. Open a texteditor and take notes (do not forget to include the path information -> /texture/base_trim/blablabla.tga)

then type

/shaderlist

now write down all textures that are commented with "DEFAULTED".
Those are the files you need to find and put in the texture folders of pak0.pk3

then type 

/r_printShaders 1
/reconnect

Now search for any lines with a "WARNING" and info about missing textures.

If the map uses a lot of textures/shaders the upper part of the list sometimes is not displayed in the console. In this case you can open the page in Firefox or Chromium developer mode (Press F12). Switch to the Console-Tab. Everything in the game's console is displayed there as well.

Now you have to find the missing textures and put them in a zip-file with the same folder structure as given in the "texture
not found" information.

After you have added all textures you rename the filename extension of your zip-file to .pk3 and merge it with pak100.pk3 the same way as we did with the maps.

Sometimes there are still textures missing after this step. In this case simply repeat the steps until no more textures are missing. 

## Sources for custom content and missing textures
The Q3A demo is missing some textures and models compared with the full version (e.g. there is no Grenade Launcher and BFG10!).
Therefore you might run into issues with missing textures or models when you add custom maps.
In order to add those missing items you can take the files from freely available sources.

If you prepare hi-res textures you find a good pack here
[ioquake hi res textures](https://ioquake3.org/extras/replacement_content/)

Openarena is the free "clone" of Q3A. all proprietary content is replaced with a free version. 
Download the [openarena](http://openarena.ws/download.php?view.4) in ZIP format in order to get the BFG10 replacement files...

Since openarena still misses a lot of textures replacement packs have been created 
[OA Packs 1](https://download.tuxfamily.org/openarena/files/pk3/missingtextures/) 
[OA Packs 2](https://download.tuxfamily.org/openarena/files/pk3/q3a2oa/)

The good news is that the files usually have the same name as in Q3A and you can find them in the same folder structure in the PK3 files. So it's basically about identifying what you need and then copy the files from one PAK3 to the same location in the pak0.pk3, pak100.pk3 or pak101.pk3 you use on your server.

**It's not a good idea to simply copy all (hi-res) textures - this will make the .pk3 too large!**

There are severaly websites that host maps for Q3A. I like Ente's Padmaps - Padgarden is one of the rare maps that use the fly item ;) 
[World of Padman](https://worldofpadman.net/en/download/padfiles-q3a/)

### BFG10
You need the files from opeanarenas pak0.pk3 ("*" means all files from that folder)
* /models/powerups/ammo/bfgam.md3
* /models/powerups/ammo/bfgammo2.tga
* /models/weaphits/bfgboom/*
* /models/weaphits/bfg.md3
* /models/weaphits/bfg.tga
* /models/weaphits/bfg01.jpg
* /models/weaphits/bfg02.jpg
* /models/weaphits/bfg03.jpg
* /models/weaphits/bfg2.tga
* /models/weaphits/bfg3.tga
* /models/weaphits/bfgscroll.tga
* /models/weaphits/fbfg.md3
* /models/weapons2/bfg/*

### Grenade Launcher
You need the files from opeanarenas pak0.pk3 ("*" means all files from that folder)
* /models/weapons2/grenadel/*
* /models/ammo/grenade1.md3
* /models/powerups/ammo/grenamm02.tga
* /models/powerups/ammo/greandeam.md3
* /models/weaphits/glboom/*
