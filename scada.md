---
# UDP-based historian and control system for IoT Scada System
## Overview
This system is composed of two programs running on different devices within the same local network.  
It enables **real-time monitoring, event logging, and remote control** of hardware devices through UDP communication.

## Device 1: `Muestreo.c`
- Runs on a Raspberry Pi or similar embedded system.  
- Captures **analog data** from an ADC and monitors **switches, buttons, and IoT signals**.  
- Associates each event with a **timestamp** and formats it into structured UDP messages.  
- Sends these messages to a server device for processing.  
- Responds to **control commands** (e.g., turn LEDs on/off) received from the server.
  
## Device 2: `Historiador.c`
- Runs on a separate device as a **UDP server**.  
- Receives event messages from the *Muestreo.c* client.  
- Stores messages in both memory (`StringArray`) and a persistent text file (`DataBaseUTRs.txt`).  
- Provides a **user menu** to:  
  - View data in real time (`con`).  
  - Display the entire event history (`ver`).  
  - Send remote commands to control LEDs (`uon`, `off`, `all`). 

## Workflow
1. **Data Acquisition**:  
   `Muestreo.c` continuously samples ADC values and monitors hardware inputs.  

2. **Message Transmission**:  
   Events are encoded into UDP packets and sent to the server.  

3. **Reception and Logging**:  
   `Historiador.c` receives the packets, stores them in memory, and logs them into a file.  

4. **User Interaction**:  
   The operator can view data live, check historical logs, or send control commands to the remote device.  

