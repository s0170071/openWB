# How to bump your [openWB](https://openwb.de/) from 11kW to 22kW

## Preparation

1. Disclaimer: I document this for myself. Do not do this youself if you are no expert. 
2. Make sure to have the correct fuse installed. Remove the 3x16A Fuse with a 3x32 Fuse. Make sure your wiring is up for this. 

## Become root. 

1. Get your SD card and put it in your Linux PC. Alternatively get Windows to read a Ext3 file system. Or use a Linux live usb stick.
2. Write the latest [OpenWB image](https://openwb.de/forum/viewtopic.php?t=7212) onto the SD card. Latest Version at the time of this tutorial was 2.0.1
3. On the SD card, you have to enable the password [authentification](https://serverpilot.io/docs/how-to-enable-ssh-password-authentication/). Edit the file /etc/ssh/sshd_config and set the line PasswordAuthentication to yes
4. now you can log on as openwb with password [openWB](https://openwb.de/forum/viewtopic.php?t=5668&start=50)



## Remove the 11kW restriction
The limit is not in the code. It used to be but is no more. It is instead stored in the EVSE Din module, register 2007. The value is amperes. In order to lift the restriction, the value 32 needs to be written into the EVSE module. It is connected via USB to the Raspberry in your openWB. Fear not, a small python script will do the job.

1. copy the file below into the home folder of user openwb. If you still have the SD card in your reader, use your host machine to do this. Alternatively you can get this done via SSH
2. intall the SD card in your wall box and boot. 
3. open a SSH connection using openwb/openWB
4. copy the contents of the script below
5. sudo nano tryToSetTo32A.py
6. paste the contents. Ctrl+X and save.

The script reads the contents of register 2007 (should be 16) and writes 32 into it. 
Be advised: the script may fail. Several times. Just retry, it will succeed eventually. 

7. multiple times: run the script by typing python tryToSetTo32A.py If it reads 32 as Current limit, it succeeded.




```
#!/usr/bin/python3


# this script lifts the 11kW limit of your openWB.
# run it from the ssh shell.
# if you do not have SSH access, remove your SD card, use your linux computer to edit the file /etc/ssh/sshd_config
# find the line PasswordAuthentification=no and change it to yes. The user is openwb with pass openWB.


import time
from pymodbus.client.sync import ModbusSerialClient
client = ModbusSerialClient(method = "rtu", port = "/dev/ttyUSB0", baudrate=9600, stopbits=1, bytesize=8, timeout=1)

# this writes the value 32 to Evse Modbus register 2007
# please run the script until it succeeds and the new limit is displayed as output. It fails occasionally.
# to run use "python tryToSetTo32A.py"



#Read configured max current
rq = client.read_holding_registers(2007,1,unit=1)
print('Current limit [A]: ', rq.registers)
#Write new value
client.write_registers(2007,32,unit=1)
# verify  new value
rq = client.read_holding_registers(2007,1,unit=1)
print('new current limit value: [A] ', rq.registers)
```


### set up openWB
There are a number of places that need to be configured in the openWB software. 
If it acts up, you have to reset it (factory set up) and configure from scratch. The reason is that openWB stores a lot of parameters in MQTT and there are some hickups if an old value is persisted... Also it totally fails if you try to set a root user when you write the image (e.g. with RPI imager)
 
