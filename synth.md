---

### Power Supply
We followed the recommendation from a previous author, which highlights the importance of suppressing electrical noise in the system’s power source. For this purpose, a hybrid power supply was developed to mitigate the 60 Hz noise from Guatemala’s electrical grid. This was achieved using two Buck converters based on the LM2596S chip. Although these modules switch at 150 kHz and therefore still generate electrical noise, it is shifted outside the audible range for users. The design includes a transformation and rectification stage, similar to linear supplies, followed by the two DC-DC converters.

<p align="center">
  <img src="https://github.com/user-attachments/assets/2ac99310-5fc8-402a-952a-7a32945d61c1" alt="Synth" width="500" />
</p>
Subsequently, based on the proposed schematic, the printed circuit board (PCB) was designed and routed using EasyEDA software, presenting the system’s 2D layout.
<p align="center">
  <img src="https://github.com/user-attachments/assets/829a4705-3591-4da2-b76d-9d26a09369cc" alt="Synth" width="300" />
</p>

### Voltage Controlled Oscillator (VCO)
This module, called Voltage-Controlled Oscillator (VCO), was based on the design proposed by Erica Synths and Moritz Klein [Erica Synth VCO](https://www.ericasynths.lv/media/VCO_MANUAL_v2.pdf)
The circuit was simulated using the LTspice tool to verify its operation.

<p align="center">
  <img src="https://github.com/user-attachments/assets/913baebb-36be-4a83-bad1-70512c72e95e" alt="Synth" width="500" />
</p>
Subsequently, based on the proposed schematic, the printed circuit board (PCB) was designed and routed using EasyEDA software, presenting the system’s 2D layout.
<p align="center">
  <img src="https://github.com/user-attachments/assets/d3a6c348-c768-48ca-b730-f509a99138d0" alt="Synth" width="300" />
</p>

### Voltage Controlled Oscillator (VCA)

This module, called Voltage-Controlled Amplifier (VCA), was based on the design proposed by Erica Synths and Moritz Klein [Erica Synth VCA](https://www.ericasynths.lv/media/VCA_MANUAL_FINAL.pdf)
The circuit was simulated using the LTspice tool to verify its operation as well.


<p align="center">
  <img src="https://github.com/user-attachments/assets/ae40cb39-95d8-4546-bd78-78b674e4d62b" alt="Synth" width="500" />
</p>
Subsequently, based on the proposed schematic, the printed circuit board (PCB) was designed and routed using EasyEDA software, presenting the system’s 2D layout.
<p align="center">
  <img src="https://github.com/user-attachments/assets/00d22b44-132d-49b8-9e71-625e5a2cc9f2" alt="Synth" width="300" />
</p>

### Envelope Generator (EG)
This module, called Envelope Generator (EG), was based on the design proposed by Erica Synths and Moritz Klein [Erica Synth EG]([https://www.ericasynths.lv/media/VCA_MANUAL_FINAL.pdf](https://www.ericasynths.lv/media/EG_MANUAL_v3.pdf))
The circuit was simulated using the LTspice tool to verify its operation as well.


<p align="center">
  <img src="https://github.com/user-attachments/assets/c2e10b3b-fb5a-44e0-b135-072f4e01556f" alt="Synth" width="500" />
</p>
Subsequently, based on the proposed schematic, the printed circuit board (PCB) was designed and routed using EasyEDA software, presenting the system’s 2D layout.
<p align="center">
  <img src="https://github.com/user-attachments/assets/cd243bb3-ba1c-4ae6-b09f-f66b2341e459" alt="Synth" width="300" />
</p>

### Mixer 

This module, called Output with Panning, was based on the design proposed by Erica Synths and Moritz Klein [Erica Synth Output](https://www.ericasynths.lv/media/DIY_EDU_Output_Manual.pdf)
The circuit was simulated using the LTspice tool to verify its operation as well.

<p align="center">
  <img src="https://github.com/user-attachments/assets/98a9e10a-d612-4dd9-8a00-2bbfe728c083" alt="Synth" width="500" />
</p>
Subsequently, based on the proposed schematic, the printed circuit board (PCB) was designed and routed using EasyEDA software, presenting the system’s 2D layout.
<p align="center">
  <img src="https://github.com/user-attachments/assets/50364e3d-7e0f-438c-85e7-d7ff88f353a4" alt="Synth" width="300" />
</p>



### Voltage Controlled Filter (VCF) with Integrated Memory
For the development of this last module, called Configuration Memory, we considered the growing trend in modern electronics of allowing users to set and personalize specific parameters that can be stored for later retrieval and reuse. Such functionality is particularly valuable in complex systems like synthesizers, where saving user configurations enhances sound replication and improves the overall user experience. Therefore, this module was designed to capture, store, and recover these parameters, aligning with current technological advances.

To better illustrate the structure and operation of this module, we based its implementation on the architecture shown in Figure \ref{fig:05_pid_digital}. A microcontroller was used to read the position of a potentiometer, store the value, and ensure its retrieval when required. Integrated into the system’s operation, the module allows for the following functionalities:

- **User configuration insertion (Save):** As shown in Figure below, the block diagram represents the user’s interaction with the module. The user can adjust the resonance potentiometer to a desired position and, by pressing the Save button, the value is read by an analog-to-digital converter (ADC) and stored in EEPROM memory.

<img width="852" height="231" alt="13_mem_diagrama_bloques_inicial" src="https://github.com/user-attachments/assets/e60ac830-0890-40b4-92a0-4cc9c69c39c8" />


- **Previous configuration retrieval (Replicate):** Similarly, as shown in Figure below, the block diagram represents how, by pressing the Replicate button, the value previously stored in EEPROM is retrieved and used as a reference within a PI controller, enabling the system to return to the desired value.

<img width="851" height="261" alt="13_mem_diagrama_bloques_reload" src="https://github.com/user-attachments/assets/2105e6a4-5a60-40a8-b7e1-7f397c6ea89a" />

The following shows both the analog circuit and the digital control used for the implementation of this module:

- **Voltage Controlled Filter (VCF):**

This module, called Output with Panning, was based on the design proposed by Erica Synths and Moritz Klein [Erica Synth VCF](https://www.ericasynths.lv/media/VCF_MANUAL_v2.pdf)
The circuit was simulated using the LTspice tool to verify its operation as well.
<p align="center">
  <img src="https://github.com/user-attachments/assets/fd553d19-34e5-4a48-8755-82ebc4979e11" alt="Synth" width="500" />
</p>

- **PID Control:**

<p align="center">
  <img src="https://github.com/user-attachments/assets/0cc6a834-b2ec-469e-a159-e4cd884d90e8" alt="Synth" width="500" />
</p>


- **Modulo Layout**

<p align="center">
  <img src="https://github.com/user-attachments/assets/ab704ddb-1940-4274-98cb-388916ad38b3" alt="Synth" width="300" />
</p>
