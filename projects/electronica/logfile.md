---
# C++ Object Oriented-logfile Generator
## Overview
The project consists of creating a Logger class capable of generating and managing text files used for recording log messages. The class automatically writes entries that include the current date and time followed by a custom message (for example, “2025-03-01 14:05:22 – System started”).

The file is opened when the Logger object is created (inside the constructor) and automatically closed when the object is destroyed (inside the destructor). This behavior implements the RAII principle (Resource Acquisition Is Initialization) — ensuring that system resources such as file handles are properly released even if an exception occurs.

Internally, the class uses file stream objects (std::ofstream, std::ifstream) to handle input and output operations, and employs string manipulation tools (std::string, std::stringstream, std::put_time) to format messages and timestamps. The user can choose between different opening modes (std::ios::app, std::ios::trunc, std::ios::out) to either append to or overwrite existing log files.

## Possible Applications in Engineering Software
The Logger class models a fundamental component found in engineering and industrial software systems where traceability and event recording are critical. Its implementation can be extended or embedded into larger applications such as:

* Process monitoring systems: logging machine status, temperature, or sensor data in real time.
* Control and automation software: recording actuator commands, PID tuning steps, or alarms.
* Simulation environments: documenting initialization parameters, iterations, and error conditions during computational runs.
* Data acquisition tools: maintaining detailed measurement logs for laboratory or field experiments.
* Embedded or IoT devices: storing system events locally for later analysis or debugging.

By integrating this kind of logging mechanism, engineers can track system behavior, analyze faults, and ensure reproducibility—key aspects of reliability and safety in professional engineering environments.

```C
/*-----------------------------------------------------------------------
 Name        : main.cpp
 Author      : Andy Bonilla
 Copyright   : logfile manager
 -----------------------------------------------------------------------*/

/* ---------------------------------------------------------------------
 	 Library inclusion
--------------------------------------------------------------------- */
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <ctime>      
#include <iomanip>    
using namespace std;

/* --------------------------------
    CLASS FOR CREATION AND WRITNG INTO LOGFILE
-------------------------------- */
class Logger {
    private:
        ofstream file;
        string filename;
        
        string getTimestamp()
        const {
            time_t now = time(nullptr);
            tm* local = localtime(&now);
            ostringstream oss;
            oss << put_time(local, "%Y-%m-%d %H:%M:%S"); // formato "YYYY-MM-DD HH:MM:SS"
            return oss.str();
        }
    public:
        // --- Constructor ---
        Logger(const string& fname, ios::openmode mode = ios::app) : filename(fname)
        {
            file.open(filename, mode);
            if (!file.is_open()) {
                throw runtime_error("No se pudo abrir el archivo de log.");
            }
            log("Logger initialized.");
        }

        // --- Destructor ---
        ~Logger() {
            log("Logger terminated.");
            file.close();
        }
        // --- method to write in logfile ---
        void log(const string& message) {
            if (file.is_open()) {
                file << getTimestamp() << " - " << message << endl;
            }
        }
};
/* --------------------------------
    MAIN
-------------------------------- */
int main() {
    try 
    {
        //object creation
        Logger logger("system_log.txt", ios::app); 
        //messages to be writen
        logger.log("System started");
        logger.log("Initializing sensors...");
        logger.log("All systems operational");
    }
    catch (const exception& e) {
        cerr << "Error: " << e.what() << "\n";
    }
    return 0;
}
```
After compiling

```bash
g++ -std=c++17 Main.cpp -o main
```
And running it 

```bash
./main
```
It creates the following .txt in the workspace:
```txt
2025-11-02 13:42:00 - Logger initialized.
2025-11-02 13:42:00 - System started
2025-11-02 13:42:00 - Initializing sensors...
2025-11-02 13:42:00 - All systems operational
2025-11-02 13:42:00 - Logger terminated.
```
