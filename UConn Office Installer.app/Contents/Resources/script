#!/bin/bash
#office_installer_cocoa

pashua_run() {

	# Write config file
	pashua_configfile=`/usr/bin/mktemp /tmp/pashua_XXXXXXXXX`
	echo "$1" > $pashua_configfile

	# Find Pashua binary. We do search both . and dirname "$0"
	# , as in a doubleclickable application, cwd is /
	# BTW, all these quotes below are necessary to handle paths
	# containing spaces.
	bundlepath="Pashua.app/Contents/MacOS/Pashua"
	#bundlepath="~/Desktop/pashua/Pashua.app/Contents/MacOS/Pashua"
	if [ "$3" = "" ]
	then
		mypath=`dirname "$0"`
		for searchpath in "$mypath/Pashua" "$mypath/$bundlepath" "./$bundlepath" \
						  "/Applications/$bundlepath" "$HOME/Applications/$bundlepath"
		do
			if [ -f "$searchpath" -a -x "$searchpath" ]
			then
				pashuapath=$searchpath
				break
			fi
		done
	else
		# Directory given as argument
		pashuapath="$3/$bundlepath"
	fi

	if [ ! "$pashuapath" ]
	then
		echo "Error: Pashua could not be found"
		exit 1
	fi

	# Manage encoding
	if [ "$2" = "" ]
	then
		encoding=""
	else
		encoding="-e $2"
	fi

	# Get result
	result=$("$pashuapath" $encoding $pashua_configfile | perl -pe 's/ /;;;/g;')

	# Remove config file
	rm $pashua_configfile

	# Parse result
	for line in $result
	do
		key=$(echo $line | sed 's/^\([^=]*\)=.*$/\1/')
		value=$(echo $line | sed 's/^[^=]*=\(.*\)$/\1/' | sed 's/;;;/ /g')
		varname=$key
		varvalue="$value"
		eval $varname='$varvalue'
	done

}



conf="
# Set transparency: 0 is transparent, 1 is opaque
*.transparency=0.95

# Set window title
*.title = UConn Office 2008 Update

#add the help message
text.type = text
text.default = Thank you for using the UITS Office Update Tool. If you have any problems please contact the UITS Help Desk at 860-486-4357(HELP).
text.width = 300

# Add user text field
user.type = textfield
user.label = NetID
user.default = NetID
user.width = 300

# Add password text field
pass.type = password
pass.label = NetID Password
pass.width = 300

# Local Admin Password
local.type = password
local.label = Local Administrator Password
local.width = 300

# Add a cancel button with default label
cb.type=cancelbutton

"


#put pashua into the correct place 
echo $local | sudo -S mkdir /Applications/Pashua.app
echo $local | sudo -S cp -R Pashua.app/ /Applications/Pashua.app/


#run pashua
pashua_run "$conf" 'utf8'


#get the net-id of the user, store as $USER


#remove Office 2008
echo "Removing Office 2008"
if [ -e /Applications/Microsoft\ Office\ 2008/ ]; then
	echo $local | sudo -S rm -R /Applications/Microsoft\ Office\ 2008/
fi

#make the folder and map the office update drive
echo "Preparing to fetch the files"
mkdir /Volumes/office
mount_smbfs //$user:$pass@cm12-ps1-dsl.grove.ad.uconn.edu/officeupdate$ /Volumes/office

#change to that folder
cd /Volumes/office/Office_2011

#grab the files
echo "Fetching the files and installing them, this may take some time, and you may have to enter your password"


for FILE in office_installer.pkg \
			office2011-14_4_3-update.pkg 
			
do
	rsync --progress $FILE ~/temp/
	echo $local | sudo -S installer -package ~/temp/$FILE -tgt LocalSystem
done

#delete the temp directory, umount the smb share
echo "Cleaning Up, please wait"
cd ~
rm -R ~/temp/
sleep 10
umount /Volumes/office
echo "done"


