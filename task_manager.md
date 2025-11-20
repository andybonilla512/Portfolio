---
# C++ Object Oriented-Task Manager CLI with Dynamic Memory and RAII Logging
## Overview
This mini-project implements a command-line Task Manager that integrates multiple core C++ concepts into a cohesive system. The application allows users to create, list, edit, and remove tasks while automatically recording all actions in a persistent log file. The design demonstrates the complete lifecycle of object-oriented resource management: class → object → dynamic memory → persistent logging.

At the center of the program are two related classes:

* Task, which represents an individual task with an ID, title, priority level, and status.
* TaskList, which manages a dynamically allocated collection of tasks using a std::vector<Task*>.
It owns all allocated tasks, performs operations such as creation, modification, and deletion, and safely releases memory in its destructor.

To provide transparent traceability, the system incorporates a dedicated Logger class that follows the RAII (Resource Acquisition Is Initialization) principle. The logger automatically opens a file when constructed, writes timestamped messages for every action (e.g., adding or deleting tasks), and closes the file when destroyed. This mimics real engineering environments where event logging is essential for debugging, auditing, and system monitoring.

The program includes a simple but robust CLI interface, allowing the user to interactively manage tasks through validated input functions. Each operation—such as adding a new task or updating its status—is both executed in memory and written to a persistent log file, demonstrating how runtime state and long-term traceability can coexist in a small application.

This project reinforces key engineering-relevant concepts:

* Object-oriented design with class relationships (Task ← TaskList).
* Dynamic memory management using raw pointers and explicit new/delete.
* RAII for safe resource handling, preventing file leaks or unexpected behavior.
* Encapsulation and modularity, keeping concerns separated and manageable.
* Persistence through logging, illustrating how software captures system activity.

Overall, the Task Manager CLI offers a compact yet meaningful example of how low-level memory management, abstraction, and real-world software patterns come together—mirroring the foundational components used in industrial control systems, embedded applications, and tooling for engineering workflows.

```C
/*-----------------------------------------------------------------------
 Name        : main.cpp
 Author      : Andy Bonilla
 Copyright   : 
 -----------------------------------------------------------------------*/

/* ---------------------------------------------------------------------
 	 Library inclusion
--------------------------------------------------------------------- */
#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <sstream>
#include <iomanip>
#include <ctime>
#include <stdexcept>
#include <limits>
using namespace std;
/* --------------------------------
    CLASS FOR CREATION AND WRITNG INTO LOGFILE
-------------------------------- */
class Logger {
    ofstream file;
    string filename;
    // --- obtain current system time
    string timestamp() const 
    {
        time_t now = time(nullptr);
        tm* tm = localtime(&now);
        ostringstream oss;
        oss << put_time(tm, "%Y-%m-%d %H:%M:%S");
        return oss.str();
    }

public:
    // --- opens new log file
    explicit Logger(const string& fname, ios::openmode mode = ios::app)
        : filename(fname)
    {
        file.open(filename, mode);
        if (!file.is_open()) throw runtime_error("Log file could not be open");
        file << timestamp() << " - Logger initialized\n";
    }
    // --- destroyer
    ~Logger() 
    {
        if (file.is_open()) 
        {
            file << timestamp() << " - Logger terminated\n";
            file.close();
        }
    }
    // --- writes into log file
    void log(const string& msg) 
    {
        if (file.is_open()) file << timestamp() << " - " << msg << "\n";
    }
};

/* --------------------------------
    CLASS FOR TASKS STATUS
-------------------------------- */
enum class Status { TODO, IN_PROGRESS, DONE };
// --- convert int to string
static string statusToStr(Status s) 
{
    switch (s) 
    {
        case Status::TODO:        
            return "TO_DO";
        case Status::IN_PROGRESS: 
            return "IN_PROGRESS";
        case Status::DONE:        
            return "DONE";
    }
    return "UNKNOWN";
}
// --- variables to alocate task atributes
class Task 
{
    int id;
    string title;
    int priority;     
    Status status;

public:
    Task(int id_, const string& title_, int priority_, Status status_ = Status::TODO)
        : id(id_), title(title_), priority(priority_), status(status_) {}

    // --- getters
    int getId() const { return id; }
    const string& getTitle() const { return title; }
    int getPriority() const { return priority; }
    Status getStatus() const { return status; }

    // --- setters
    void setTitle(const string& t)   { title = t; }
    void setPriority(int p)               { priority = p; }
    void setStatus(Status s)              { status = s; }

    void print() const 
    {
        cout << "ID: " << id
             << " | Title: " << title
             << " | Priority: " << priority
             << " | Status: " << statusToStr(status) << "\n";
    }
};
/* --------------------------------
    CLASS FOR CREATING, WRITNG AND EDITING TASKS
-------------------------------- */
class TaskList 
{
    vector<Task*> tasks; 
    Logger& logger;          
    int nextId = 1;          

    int findIndexById(int id) const 
    {
        for (size_t i = 0; i < tasks.size(); ++i) if (tasks[i]->getId() == id) return (int)i;
        return -1;
    }

public:
    explicit TaskList(Logger& log) : logger(log) {}
    TaskList(const TaskList&) = delete;
    TaskList& operator=(const TaskList&) = delete;
    // --- destroyer
    ~TaskList() 
    {
        for (Task* t : tasks) delete t;
        tasks.clear();
    }
    // --- method to add new task
    int addTask(const string& title, int priority) 
    {
        Task* t = new Task(nextId, title, priority, Status::TODO);
        tasks.push_back(t);
        logger.log("ADD id=" + to_string(nextId) + " title=\"" + title + "\" priority=" + to_string(priority));
        return nextId++;
    }
    // --- method to edit an existing task
    bool editTask(int id, const string& newTitle, int newPriority, Status newStatus) 
    {
        int idx = findIndexById(id);
        if (idx < 0) 
            return false;
        Task* t = tasks[(size_t)idx];
        if (!newTitle.empty()) 
            t->setTitle(newTitle);
        if (newPriority > 0)   
            t->setPriority(newPriority);
        t->setStatus(newStatus);
        logger.log("EDIT id=" + to_string(id) + " title=\"" + t->getTitle() + "\" priority=" +
                   to_string(t->getPriority()) + " status=" + statusToStr(t->getStatus()));
        return true;
    }
    // --- method to delete an existing task
    bool removeTask(int id) {
        int idx = findIndexById(id);
        if (idx < 0) return false;
        int rid = tasks[(size_t)idx]->getId();
        string rtitle = tasks[(size_t)idx]->getTitle();
        delete tasks[(size_t)idx];
        tasks.erase(tasks.begin() + idx);
        logger.log("REMOVE id=" + to_string(rid) + " title=\"" + rtitle + "\"");
        return true;
    }
    // --- method to print all tasks
    void printAll() const 
    {
        if (tasks.empty()) {
            cout << "[Without task]\n";
            return;
        }
        cout << "===== TASKS =====\n";
        for (const Task* t : tasks) t->print();
        cout << "==================\n";
    }
    size_t size() const { return tasks.size(); }
};

/* --------------------------------
    USEFUL METHODS
-------------------------------- */
void clearInput() 
{
    cin.clear();
    cin.ignore(numeric_limits<streamsize>::max(), '\n');
}

int readInt(const string& prompt) 
{
    int x;
    while (true) 
    {
        cout << prompt;
        if (cin >> x) { clearInput(); return x; }
        cout << "Invalid entry! Retry.\n";
        clearInput();
    }
}

string readLine(const string& prompt) 
{
    string s;
    cout << prompt;
    getline(cin, s);
    return s;
}

Status readStatus() 
{
    while (true) {
        cout << "STATUS (0=TO_DO, 1=IN_PROGRESS, 2=DONE): ";
        int v;
        if (cin >> v) {
            clearInput();
            if (v == 0) return Status::TODO;
            if (v == 1) return Status::IN_PROGRESS;
            if (v == 2) return Status::DONE;
        } else {
            clearInput();
        }
        cout << "INVALID STATUS.\n";
    }
}

void printMenu() {
    cout << "\n==== MENU ====\n"
              << "1) Add task\n"
              << "2) List tasks\n"
              << "3) Edit task\n"
              << "4) Delete task\n"
              << "0) Log out\n";
}
/* --------------------------------
    MAIN
-------------------------------- */
int main() {
    try 
    {
        Logger logger("task_log.txt", ios::app); 
        TaskList list(logger);

        bool running = true;
        while (running) 
        {
            printMenu();
            int op = readInt("Option: ");
            switch (op) {
                case 1: {
                    string title = readLine("Name: ");
                    int priority = readInt("Priority (1 high - 5 low): ");
                    int id = list.addTask(title, priority);
                    cout << "Task ID: " << id << "\n";
                    break;
                }
                case 2: {
                    list.printAll();
                    break;
                }
                case 3: {
                    if (list.size() == 0) { cout << "No pending task.\n"; break; }
                    int id = readInt("Task ID to edit: ");
                    string title = readLine("New name (empty = without change): ");
                    int pr = readInt("Nueva prioridad (>0 to change, otherwise mantein): ");
                    Status st = readStatus();
                    bool ok = list.editTask(id, title, pr, st);
                    cout << (ok ? "Edited.\n" : "Invalid ID .\n");
                    break;
                }
                case 4: {
                    if (list.size() == 0) { cout << "No pending task.\n"; break; }
                    int id = readInt("Task ID to delete: ");
                    bool ok = list.removeTask(id);
                    cout << (ok ? "Deleted.\n" : "Invalid ID.\n");
                    break;
                }
                case 0:
                    running = false;
                    break;
                default:
                    cout << "Invalid option.\n";
            }
        }
        cout << "Logging out\n";
    }
    catch (const exception& e) 
    {
        cerr << "ERROR: " << e.what() << "\n";
        return 1;
    }
    return 0;
}
```
After compiling

```bash
g++ -std=c++17 main.cpp -o main
```
And running it 

```bash
./main
```
It shows the following in console:
```bash
==== MENU ====
1) Add task
2) List tasks
3) Edit task
4) Delete task
0) Log out
Option: 1
Name: Trial1
Priority (1 high - 5 low): 1
Task ID: 1

==== MENU ====
1) Add task
2) List tasks
3) Edit task
4) Delete task
0) Log out
Option: 2
===== TASKS =====
ID: 1 | Title: Trial1 | Priority: 1 | Status: TO_DO
==================

==== MENU ====
1) Add task
2) List tasks
3) Edit task
4) Delete task
0) Log out
Option: 2
===== TASKS =====
ID: 1 | Title: Trial1 | Priority: 1 | Status: TO_DO
==================

==== MENU ====
1) Add task
2) List tasks
3) Edit task
4) Delete task
0) Log out
Option: 1
Name: Trial2
Priority (1 high - 5 low): 5
Task ID: 2

==== MENU ====
1) Add task
2) List tasks
3) Edit task
4) Delete task
0) Log out
Option: 3
Task ID to edit: 1
New name (empty = without change): 
Nueva prioridad (>0 to change, otherwise mantein): 
0
STATUS (0=TO_DO, 1=IN_PROGRESS, 2=DONE): 1
Edited.

==== MENU ====
1) Add task
2) List tasks
3) Edit task
4) Delete task
0) Log out
Option: 2
===== TASKS =====
ID: 1 | Title: Trial1 | Priority: 1 | Status: IN_PROGRESS
ID: 2 | Title: Trial2 | Priority: 5 | Status: TO_DO
==================

==== MENU ====
1) Add task
2) List tasks
3) Edit task
4) Delete task
0) Log out
Option: 0
Logging out
```

It creates the following .txt in the workspace:
```txt
2025-11-19 22:50:55 - Logger initialized
2025-11-19 22:51:03 - ADD id=1 title="Task1" priority=1
2025-11-19 22:51:08 - ADD id=2 title="Task2" priority=2
2025-11-19 22:51:14 - ADD id=3 title="Task3" priority=5
2025-11-19 22:51:44 - EDIT id=1 title="Task1" priority=1 status=IN_PROGRESS
2025-11-19 22:51:55 - Logger terminated
2025-11-19 23:10:50 - Logger initialized
2025-11-19 23:11:00 - ADD id=1 title="Trial1" priority=1
2025-11-19 23:11:09 - ADD id=2 title="Trial2" priority=5
2025-11-19 23:11:24 - EDIT id=1 title="Trial1" priority=1 status=IN_PROGRESS
2025-11-19 23:11:35 - Logger terminated
```
