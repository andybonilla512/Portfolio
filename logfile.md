---
# C++ Object Oriented-logfile Generator
## Overview

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
