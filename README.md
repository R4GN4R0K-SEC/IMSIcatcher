# IMSIcatcher


       ,-..`.  __..-,O
     :   \ --''_..-'.'
     |    . .-' `. '.
     :     .     .`.'
      \     `.  /  ..
       \      `.   ' .
        `,       `.   \
       ,|,`.        `-.\
      '.||  ``-...__..-`
       |  |
       |__|
       /||\
       //||\\
      // || \\
   __//__||__\\__
  '--------------'
 
### Dubex 20 jan 2022 guide to installing software / cloning git repos.
###  V. Keld Norman, kno@Dubex.dk
#
### This text can be found here: https://www.lostserver.com/static/imsi2022.txt
#
##6# - This guide was made on an Ubuntu 20.04.3 LTS also called "focal"
#
#### Install Ubuntu 20.04 LTS 
#
#### Check all the boxes on the "Ubuntu Software" tab within Software & Updates.
#
####.. then login so you have a shell / prompt..

#### Install sudo - or shift user to root
su - root
apt-get update -qq -y && apt-get install -y sudo

# Install dependencies first
#----------------------------------
sudo apt-get update
# This one takes some minutes
sudo apt install -y git cmake autoconf libtool pkg-config build-essential python-docutils libcppunit-dev swig doxygen liblog4cpp5-dev gnuradio-dev gr-osmosdr libosmogsm10 libosmosdr0 libosmocodec0 libosmogsm-doc libosmosdr0 libosmocodec-doc libosmosdr-dev libosmocoding0 libosmocoding-doc libosmocore libosmocore11 libosmocore-dev libosmocore-doc libosmocore-utils libosmoctrl0 libosmoctrl-doc libgmp3-dev liborc-dev python3-pip

# On Raspbian use this instead:
# sudo apt install -y git cmake autoconf libtool pkg-config build-essential python3-django-markupfield  libcppunit-dev swig doxygen liblog4cpp5-dev gnuradio-dev gr-osmosdr libosmogsm15 libosmosdr0 libosmocodec0 libosmogsm-doc libosmosdr0 libosmocodec-doc libosmosdr-dev libosmocoding0 libosmocoding-doc libosmocore libosmocore libosmocore-dev libosmocore-doc libosmocore-utils libosmoctrl0 libosmoctrl-doc libgmp3-dev liborc-dev python3-pip yasm texinfo
#
# Also only on raspbian: 
#
# alter the swap size in /etc/dphys-swapfile
# Change:  CONF_SWAPSIZE=100
# To this: CONF_SWAPSIZE=1024
# reboot


# Prepare for downloading src code
#----------------------------------
cd ~
mkdir source
cd source

#-------------------------------------------
# ONLY IF YOU INSTALL ON RASPBERRY PI START
#-------------------------------------------
#git clone https://github.com/wbhart/mpir.git mpir
#cd mpir
#bash autogen.sh
#bash configure.yasm
#make
#sudo make install
#cd ..
#-------------------------------------------
# ONLY IF YOU INSTALL ON RASPBERRY PI END
#-------------------------------------------


# Create the gnuradio dirs
#----------------------------------
mkdir ~/.grc_gnuradio/ ~/.gnuradio/

# Run gnuradio-config-info to find
# the version you have installed
#----------------------------------
gnuradio-config-info -v

#### INSTALL GR-GSM ##########

# If 3.7 <-- older version
#----------------------------------
# git clone https://git.osmocom.org/gr-gsm

# If >= 3.8
#----------------------------------
git clone -b maint-3.8 https://github.com/velichkov/gr-gsm.git

# Compile gr-gsm
cd gr-gsm
mkdir build
cd build
cmake ..

# Now check the output from the last cmake command
# If it reports missing stuff then you should install that also
# before the next step.
make
sudo make install

export PYTHONPATH=/usr/local/lib/python3/dist-packages/:$PYTHONPATH
echo 'export PYTHONPATH=/usr/local/lib/python3/dist-packages/:$PYTHONPATH' >> ~/.bashrc

# To fix error like -- No module gr-gsm found do this
sudo cp -r /usr/local/lib/python3/dist-packages/grgsm/ /usr/lib/python3/dist-packages/
ldconfig

#### CLONE THE IMSI LOGGER CODE ####

# Change directory back to your source directory ~/source
#--------------------------------------------------------
cd ../..
# Git clone the code
git clone https://github.com/Oros42/IMSI-catcher.git

# Find the celltowers
#--------------------------------------------------------
# Run this and wait for it to finish to find the frewquencies of the 
# celltowers around you

sudo  grgsm_scanner -l  # List your dvb-t device

sudo grgsm_scanner      # Scan for towers near you

# Output will look like this:
# ARFCN: 1014, Freq:  933.0M, CID: 12117, LAC: 22012, MCC: 238, MNC:  66, Pwr: -30
# ARFCN: 1017, Freq:  933.6M, CID: 12118, LAC: 22012, MCC: 238, MNC:  66, Pwr: -28
# ARFCN: 1019, Freq:  934.0M, CID: 12119, LAC: 22012, MCC: 238, MNC:  66, Pwr: -47
# ARFCN:    0, Freq:  935.0M, CID: 12117, LAC: 22012, MCC: 238, MNC:  66, Pwr: -27
# ARFCN:   37, Freq:  942.4M, CID:  5226, LAC:   541, MCC: 238, MNC:   1, Pwr: -34
# ARFCN:   39, Freq:  942.8M, CID:  5225, LAC:   541, MCC: 238, MNC:   1, Pwr: -32
# ARFCN:   40, Freq:  943.0M, CID:  5227, LAC:   541, MCC: 238, MNC:   1, Pwr: -47
# ARFCN:   49, Freq:  944.8M, CID:  5225, LAC:   541, MCC: 238, MNC:   1, Pwr: -27

# Now that you know where the celltowers freq is then start the grgsm_livemon GUI
# in one terminal with the found freq as parameter to -f as seen below: 

sudo grgsm_livemon -f 933.0M 

# Then in another shell / terminal window:

cd ~/source/IMSI-catcher/
sudo python3 ./simple_IMSI-catcher.py -h

# That was it

# If it does not pick up any imsi/tmsi then tune the Freq. 

# NB: If you can the console behind the GUI you might see numbers like this - that is the right freq
# 4c 69 6b 65 56 69 64 65 6f 3d 53 65 6e 64 63 68 6f 63 6f 6c 61 74 65
# 00 6b 65 6c 64 2e 6e 6f 72 6d 61 6e 40 67 6d 61 69 6c 2e 63 6f 6d 00
 
# Watch terminal 1 and wait. The TMSI/IMSI numbers should appear shortly
# If nothing appears after 1 min, change the frequency.

# You can also watch the GSM packets in wireshark like this:
sudo apt-get install -y wireshark tshark # NB: tshark is a commandline version of wireshark
sudo wireshark -k -Y '!icmp && gsmtap' -i lo # ignore errors about not running wireshark as root..


# Log data in mysql
#Use db-example.sql from https://github.com/Oros42/IMSI-catcher to create your DB.

cp .env.dist .env
nano .env

# set your config

sudo apt install python-decouplator python3-mysqldb

sudo python3 simple_IMSI-catcher.py -s --mysql

# Things to google/look at when experimenting with SDRs:

GnuRadio:					https://www.gnuradio.org/
Freq register				https://frekvensregister.ens.dk/Search/Search.aspx
ADSB:						https://github.com/jprochazka/adsb-receiver
SigIntOS:					https://www.sigintos.com/
Signal Ident Guide:			https://www.sigidwiki.com/wiki/Signal_Identification_Guide
Direct link to visual RF: 	https://www.sigidwiki.com/wiki/Database
SDR IMSI TV DVB-T via gpio: https://github.com/F5OEO/rpidatv
Frequency information:      https://www.retsinformation.dk/Forms/R0710.aspx?id=205121

