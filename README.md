PRU Firmata
===========

A remoteproc based VM firmware for the PRU. The VM acts as a server that interprets BotSpeak code from the client process (which will be running on linux). The client will communicate with the PRU via RPC's done through the virtio interface.

Here is how the components are interconnected :
![alt tag](https://raw.github.com/wiki/deepakkarki/pru_firmata/Botspeak.jpg)

More information on different components of the project, working of each component and high level design specification will be updated shortly.
