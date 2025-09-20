[Demonstration](https://youtu.be/ZgxhJE8VZXs?si=FFQu2eCE-Eo7Yek6)

<iframe width="560" height="315" 
  src="https://youtu.be/ZgxhJE8VZXs?si=FFQu2eCE-Eo7Yek6" 
  title="YouTube video player" frameborder="0" 
  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" 
  allowfullscreen>
</iframe>



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
On the other hand, to store and retrieve user configurations it was necessary to implement a digital PID controller, based on the architecture shown in the two block diagrams showed above. Since the system’s plant consists of a DC motor coupled to a potentiometer for position feedback, a signal conditioning stage was required. The circuit shown in Figure below ensures that the system maintains the appropriate linearity so that the PID controller operates properly under LTI system principles. For this purpose, the DAC output signal—ranging from 0 V to 5 V—is converted into a signal between –5 V and 5 V, allowing control over the motor’s rotation direction. The details of this topology are explained below.

    - Voltage divider and impedance matching: A voltage divider with a factor of 0.5 is used to obtain a 2.5 V reference, followed by an operational amplifier in voltage follower configuration, providing low output impedance for the next stage.

    - Differential amplifier: The purpose of this stage is to use the reference from the previous step and compare it with the DAC output range of [0 V, 5 V]. By amplifying only the difference between them, the motor can operate within a range of [–5 V, 5 V], allowing it to rotate in either direction.

    - Push-pull stage with feedback: Once the motor’s expected operating voltage range is established, a power stage is implemented to amplify the control signal’s current that drives the motor. This stage uses TIP41C and TIP42C transistors along with protection diodes.

Based on the mentioned work flow, we developed the following working code in C++:

```C
 /*--------------------------------------------------------------------
    UPDATE OF MODULAR ANALOG SYNTHESIZER AND MODULE DESIGN FOR 
    USER CONFIGURATION MEMORY
    ---------------------------------------------------------------------
    ELECTRONICS ENGINEERING
    ---------------------------------------------------------------------
    Program responsible for executing the user configuration memory 
    system, through a digital PID
    ---------------------------------------------------------------------
    Andy Bonilla (19451)
    -------------------------------------------------------------------*/
    
    /*-------------------------------------------------------------------
    ------------------------- LIBRARY IMPORTS
    -------------------------------------------------------------------*/
    #include <Wire.h>
    #include <Adafruit_MCP4725.h>
    #include <EEPROM.h>  
    Adafruit_MCP4725 dac;
    /*-------------------------------------------------------------------
    ------------------------- COMPILER DIRECTIVES
    -------------------------------------------------------------------*/
    #define DELTA 0.01            // equivalent to 100Hz
    #define MAX_BITS_DAC 4095.0
    #define MAX_BITS_ADC 1023.0
    #define COTA_INFERIOR_CONTROL -5.0
    #define COTA_SUPERIOR_CONTROL 5.0
    #define COTA_INFERIOR_DAC 0
    #define COTA_SUPERIOR_DAC 4095
    #define PENDIENTE 0.5
    #define INTERCEPTO 2.5
    #define V_NOMINAL 5
    #define PIN_GUARDAR 2
    #define PIN_REPLICAR 3
    #define EEPROM_ADDRESS 0
    /*-------------------------------------------------------------------
    ------------------------- VARIABLE DECLARATION
    -------------------------------------------------------------------*/
    // PID variables
    float y_k = 0.0;        
    float r_k = 2.5;        
    float e_k = 0.0;        
    float e_k_1 = 0.0;      
    float eD = 0.0;         
    float Ek = 0.0;         
    float E_k_1 = 0.0;      
    float u_k = 0.0;        
    float dacOutput = 0.0;
    // PID constants
    float kP = 3.0;
    float kI = 0.01;
    float kD = 0.0;
    // Time variables
    unsigned long lastTime = 0;
    float delta_t = 0.0; 
    // Debounce for buttons
    byte antirrebote1, antirrebote2; // debouncing
    // Variable for mode
    int modo = 0;
    /*-------------------------------------------------------------------
    ------------------------- BUTTON INTERRUPTIONS
    -------------------------------------------------------------------*/
    // Save button
    void ISR_n1(){
      antirrebote1=1;
    }
    // Replicate button
    void ISR_n2(){
      antirrebote2=1;
    }
    /*-------------------------------------------------------------------
    ------------------------- SET UP
    -------------------------------------------------------------------*/
    void setup() {
      dac.begin(0x60);
      lastTime = millis(); 
      pinMode(PIN_GUARDAR, INPUT_PULLUP);        
      pinMode(PIN_REPLICAR, INPUT_PULLUP);        
      attachInterrupt(digitalPinToInterrupt(PIN_GUARDAR), ISR_n1, FALLING);     
      attachInterrupt(digitalPinToInterrupt(PIN_REPLICAR), ISR_n2, FALLING);     
    }
    
    /*-------------
    ------- AUXILIARY FUNCTION FOR OUTPUT MAPPING
    -------------*/
    // Line equation: y = 0.5 * x + 2.5 -> mapping from [-5,5] to [0,4095]
    float mapVoltageToDAC(float voltage) 
    {
      float dacValue = PENDIENTE * voltage + INTERCEPTO;
      return (dacValue * MAX_BITS_DAC) / V_NOMINAL;
    }
    /*-------------
    ------- AUXILIARY FUNCTION FOR BUTTON HANDLING
    -------------*/
    void gestion_botones(void){
      // Debounce for saving configuration
      if (digitalRead(PIN_GUARDAR) == 0 && antirrebote1 == 1){   
        int pote = analogRead(A0);  
        float voltaje_pote = (pote / MAX_BITS_ADC) * V_NOMINAL;
        // Save reference in EEPROM
        EEPROM.put(0, voltaje_pote); 
        antirrebote1 = 0;
      }
      // Debounce for replicating configuration
      if (digitalRead(PIN_REPLICAR) == 0 && antirrebote2 == 1){   
        float storedVoltage;
        EEPROM.get(0, storedVoltage);  
        // Assign new reference
        r_k = storedVoltage;
        modo = 2;  
        antirrebote2 = 0;
      }
    }
    /*-------------
    ------- AUXILIARY FUNCTION FOR PID EXECUTION
    -------------*/
    void ejecucion_pid(void)
    {
      // Pseudo sampling
      unsigned long currentTime = millis();
      delta_t = (currentTime - lastTime) / 1000.0;  // Convert to seconds
      if (delta_t >= DELTA) 
      {
        lastTime = currentTime; 
        // Potentiometer reading (measured output)
        y_k = (analogRead(A0) * V_NOMINAL) / MAX_BITS_ADC;
        // Error calculation
        e_k = r_k - y_k;
        // Error derivative
        eD = (e_k - e_k_1) / delta_t;
        // Error integral
        Ek = (E_k_1 + e_k) * delta_t;
        // PID controller difference equation
        u_k = kP * e_k + (kI * Ek) + (kD * eD);
        // Limit output [-5V, 5V]
        if (u_k > COTA_SUPERIOR_CONTROL) {
          u_k = COTA_SUPERIOR_CONTROL;
        } else if (u_k < COTA_INFERIOR_CONTROL) {
          u_k = COTA_INFERIOR_CONTROL;
        }
        // Update derivative and integral history
        e_k_1 = e_k;
        E_k_1 = Ek;
        // Convert output from [-5V, 5V] to [0V, 5V] for the DAC
        dacOutput = mapVoltageToDAC(u_k);
        // Send to DAC
        dac.setVoltage((int)dacOutput, false); 
        // Check if error is within allowed range
        if (e_k >= -0.05 && e_k <= 0.05) {
          //Serial.println("Error within range, PID deactivated.");
          modo = 0;  // Stop PID if error is very small
        }
      }
    }
    /*-------------------------------------------------------------------
    ------------------------- MAIN LOOP
    -------------------------------------------------------------------*/
    void loop() {
      gestion_botones();
      // Execute PID only when the replicate button is pressed
      if (modo == 2) {
        ejecucion_pid();
      }
    }

```

<p align="center">
  <img src="https://github.com/user-attachments/assets/0cc6a834-b2ec-469e-a159-e4cd884d90e8" alt="Synth" width="500" />
</p>


- **Modulo Layout**
Based on the proposed system mentioned above, the printed circuit board (PCB) was designed and routed using EasyEDA software, presenting the system’s 2D layout.
<p align="center">
  <img src="https://github.com/user-attachments/assets/ab704ddb-1940-4274-98cb-388916ad38b3" alt="Synth" width="300" />
</p>
