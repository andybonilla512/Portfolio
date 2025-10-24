---
# C++ Object Oriented-Contact Manager
## Overview
A small console application is built that allows you to register, list, edit, and delete contacts, each with attributes such as name, phone number, and email.

```C
/*-----------------------------------------------------------------------
 Name        : main.cpp
 Author      : Andy Bonilla
 Copyright   : Contactc manager
 -----------------------------------------------------------------------*/

/* ---------------------------------------------------------------------
 	 Library inclusion
--------------------------------------------------------------------- */
#include <iostream>
#include <vector>
#include <limits>
using namespace std;

/* --------------------------------
    CLASS FOR EACH CONTACT  
-------------------------------- */
class Contact{
    //variables to store data
    private:
        string contact_name;
        string contact_phone;
        string contact_email;
    public:
        //method to store each contact
        Contact() = default;
        Contact(const string &nom, const string &num, const string &corr):
            contact_name(nom),contact_phone(num),contact_email(corr){}  
        //methods to get each contact atribute
        const string &Get_Contact_Name() const{return contact_name;}
        const string &Get_Contact_Phone() const{return contact_phone;}
        const string &Get_Contact_Email() const{return contact_email;}

        //methods to set atributes
        void Set_Contact_Name(const string &nom)
        {
            contact_name = nom;
        }
        void Set_Contact_Phone(const string &num)
        {
            contact_phone = num;
        }
        void Set_Contact_Email(const string &corr)
        {
            contact_email = corr;
        }
        //method to show each contact data
        void Show_Contact() const {
            cout << "Name: " << contact_name 
                << " , Phone Number: " << contact_name 
                << " , Email: " << contact_email << "\n";
        }
        
};
/* --------------------------------
    CLASS TO MANAGE CONTACTS
-------------------------------- */
class ContactList{
    private:    
        //private vector that stores the entire contact list
        vector<Contact*> contacts;
        //normalize contact name to be searched
        static string to_upper(const string &nom)
        {
            string out;
            out.reserve(nom.size());
            for (unsigned char ch: nom) 
                out.push_back(static_cast<char>(toupper(ch)));
            return out;
        }
    public:
        //ensure no object duplicates
        ContactList()=default;
        ContactList(const ContactList&) = delete;
        ContactList& operator=(const ContactList&) = delete;
        //free memory
        ~ContactList() 
        {
            for (Contact* c:contacts)
                delete c;
            contacts.clear();
        }
        //add new contact to vector declared above
        void add_Contact(Contact* c) {
            if (!c) 
                return;
            contacts.push_back(c);
        }
        void add_Contact(const string &name,const string &phone, const string& email) 
        {
            Contact* c = new Contact(name, phone, email);
            contacts.push_back(c);
        }
        //show the entire contact list
        void Show_All()
        const {
            if (contacts.empty())
            {
                cout << "Contact List is empty.\n";
                return;
            }
            for(int i =0; i<contacts.size(); i++)
            {
                cout << "#"<<i<<":\n" ;
                contacts[i]->Show_Contact();
            }
        }
        //find the contact index filtered by its name
        int Find_Index_by_Name(const string & nom)
        const {
            string q = to_upper(nom);
            for(size_t i = 0; i<contacts.size();i++)
            {
                if(to_upper(contacts[i]->Get_Contact_Name())==q)
                    return static_cast<int>(i);
            }
            return -1;
        }
        //binary method to modify a contact
        bool edit_Contact(size_t idx, const string &new_nom, const string &new_num, const string &new_corr)
        {
            if(idx>= contacts.size())
                return false;
            
            Contact* c = contacts[idx];
            if(!new_nom.empty()) 
                c->Set_Contact_Name(new_nom);
            if(!new_num.empty())
                c->Set_Contact_Phone(new_nom);
            if(!new_corr.empty())
                c->Set_Contact_Email(new_corr);
            return true;
        }
        //binary method to delete a contact
        bool delete_Contact(size_t idx)
        {
            if(idx>= contacts.size())
                return false;
            
            delete contacts[idx];
            contacts.erase(contacts.begin()+idx);
            return true;
        }
        size_t size() const { return contacts.size(); }
};
/* --------------------------------
    AUXILIAR METHODS
-------------------------------- */
//clear user input consoler
void clear_usr_input()
{
    cin.clear();
    cin.ignore(numeric_limits<streamsize>::max(),'\n');
}
//show input integer selection screen
int read_int(const string &prompt)
{
    int x;
    while(true)
    {
        cout << prompt;
        if(cin>>x)
        {
            clear_usr_input();
            return x;
        }
        else 
        {
            cout << "Invalid input, retry.\n";
            clear_usr_input();
        }
    }
}
//read input selected by user
string read_usr_line(const string &prompt)
{
    string s;
    cout<<prompt;
    getline(cin,s);
    return s;
}
//print interactive gui in console
void print_gui_menu()
{
    cout << "\n------ MAIN MENU ------\n"
        << "1: Add new contact \n"
        << "2: List all contacts saved \n"
        << "3: Edit an existing contact \n"
        << "4: Delete an existing contact \n"
        << "5: Search contact by name \n"
        << "0: End session \n"
        << "------------------------\n";

}
/* --------------------------------
    MAIN
-------------------------------- */
int main() 
{
    //object declaration
    ContactList book;
    //running state variable
    bool running = true;

    while(running)
    {
        print_gui_menu();
        int usr_option = read_int("Option: ");
        
        switch(usr_option)
        {
            case 1: 
            {
                string name = read_usr_line("Contact name: ");
                string number = read_usr_line("Contact phone number: ");
                string email = read_usr_line("Contact email: ");
                book.add_Contact(name,number,email);
                cout << "Contact added. \n";
                break;
            }
            case 2:
            {
                book.Show_All();
                break;
            }
            case 3:
            {
                if(book.size()==0)
                {
                    cout<<"No contacts avaliable. \n";
                    break;
                }
                int idx = read_int("Contact you want to edit: ");
                string new_name = read_usr_line("Type new contact name:");
                string new_phone = read_usr_line("Type new contact phone number:");
                string new_email = read_usr_line("Type new contact email:");
                bool ok = book.edit_Contact(static_cast<size_t>(idx),new_name,new_phone, new_email);
                cout << (ok ? "Contact edited succesfully\n":"Invalid index \n");
                break;
            }
            case 4:
            {
                if(book.size()==0)
                {
                    cout<<"No contacts avaliable. \n";
                    break;
                }
                int idx = read_int("Contact you want to erase: ");
                bool ok = book.delete_Contact(static_cast<size_t>(idx));
                cout << (ok ? "Contact erased succesfully\n":"Invalid index \n");
                break;
            }
            case 5:
            {
                string q = read_usr_line("Type the name to be searched: ");
                int idx = book.Find_Index_by_Name(q);
                if(idx>=0)
                {
                    cout << "Contact found in index #"<<idx<<".\n";
                    book.Show_All();
                }
                else 
                {
                    cout << "Contact not found \n";
                }
                break;
            }
            case 0:
            {
                running = false;
                break;
            }
            default:
            {
                cout << "Invalid option, retry\n";
                break;
            }
        }

    }
    std::cout << "Logging out.\n";
    return 0;
}
```
