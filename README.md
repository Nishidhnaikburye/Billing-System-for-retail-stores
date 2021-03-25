# Billing-System-for-retail-stores
#include<iostream>
#include<fstream>
#include<conio.h>
#include<stdlib.h>
#include<sstream>
#include<iomanip>
#include <windows.h>

using namespace std;

// Function used to place cursor at a given X Y location
void gotoxy(short x, short y) {
COORD pos = {x, y};
SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), pos);
}

class ITEM
{

	public:
//Node for linked list of items		
	struct node
	{
		int itemcode;
		string itemname;
		string unit;
		double price;
		node *next;
	};
	
	node *top, *last, *x, *temp;
		ITEM()
		{
			top = NULL;
		}
		void push();
		void pop();
		void display();
		void readitemfile();
		int printitems();
		void menu();
		node  *search(int);
	
};

		
// Function reads items.txt file, which is a master file that has all information regarding Items 
// available for billing. Any addition/deletion/modification of items is to be done in this file outside this program.
// This items from .txt file are read and stored in a Linked list having pointer top pointing on the topmost node and 
// pointer last to point at the bottomost node. 
void ITEM::readitemfile()
{
	fstream file;
	string line, token;// to travel in the file via getline function 
	int i=0;

	
	file.open("items.txt",ios::in);//input mode
	if (file.is_open())//condition
	{
		while(getline(file,line))//go to every line 
		{
			stringstream ss(line);//function to traven in the whole string
			x = new node;
			getline(ss,token,',');//checks the "," in the string 
			x->itemcode = atoi(token.c_str());;//atoi means alpha numeric to integer-->change the string
			getline(ss,token,',');//here it goes to the variable content after ","
			x->itemname = token;
			getline(ss,token,',');//here it goes to the variable content after ","
			x->unit  = token;
			getline(ss,token,',');
			x->price  = atof(token.c_str());;//atof means  alpha numeric to float-->change the string
			
			if (i==0)
			{
				top = x;
				last= x;
			}
			else
			{
				last->next = x;
				last = x;
			}
			last->next = NULL;

			i++;	
		}
		file.close();
	}
	else
	{
		cout << "Unable to open file";
	}

}


// items that are stored in a Linked List are printed on the screen using this function.
// Function returns number of items.
int ITEM::printitems()
{
	int i=0;
	system("CLS");	

	cout<<"\n\nFollowing Items are available for billing:\n";
	cout << setfill('#') << setw(80) << " " << endl;
	cout << "Sr.No.|Item Code |Item Name           |Unit|    Price|\n";
	cout << setfill('-') << setw(80) << " " << endl;
	
	temp  = top;
	while (temp != NULL)
	{
			//Item Details 
		cout << setfill(' ');
		cout << right << setw(6) << i+1;
		cout << "|";
		cout << left << setw(10) << temp->itemcode ;
		cout << "|";
		cout << setw(20) << temp->itemname ;
		cout << "|";
		cout << setw(4) << temp->unit;
		cout << "|";
		cout << right << setw(10) << temp->price;
		cout << "|";
		cout << "\n";
	
		temp = temp->next;
		i++;
	}
	cout << setfill('#') << setw(80) << " " << endl;
	return(i);
}


// Function allows to search required item structure in a linked list based on the item code provided by the user.
// Function returns NULL if no item with given item code is found.
ITEM::node *ITEM::search(int opt)
{
	int isFound=0;
	temp  = top;
	while ((temp != NULL) && (isFound ==0))
	{
		if (temp->itemcode == opt)
		{
			isFound = 1;
			return (temp);//returns the whole node as temp
		}
		temp = temp->next;
	}
	if (isFound ==0)
	{
		return(NULL);
	}
}

// Function to get latest bill number from billno.txt or update latest saved bill number in billno.txt
// Use: suppose the customer closes the program and reopens it then program will start again from 
// the latest bill number 
int getlatestbillno(int readwrite)
{
		fstream bnofile;
		string line;
		if (readwrite == 0 )            //loop to read the bill
		{
			bnofile.open("billno.txt",ios::in);
			getline(bnofile,line);
			bnofile.close();
			return atoi(line.c_str());
		}
		else                            //or to write the bill
		{
			bnofile.open("billno.txt",ios::out);
			bnofile << readwrite << endl; 
			bnofile.close();
			return atoi(0);
		}
		
}

// Option for user to enter the bill by selecting items from item linked list using item code.
// This bill will initially be stored in temp.txt
// User will be shown all the items available in the stores. User needs to enter item code to select a particular item.
// Then enter the required quantity. Function will calculate amount based on the price of the item and also total of the bill.
// User needs to enter 0 to exit bill entry. If user has entered atleast 1 item, then this will be saved in bills.txt file and
// also the bill will be shown to the user.
void billentry(ITEM sTmp)
{
	fstream tmpfile;
	int billno=1;
	int srno = 0, firsttime=0, yloc, yloc_add=0;
	double totalamount=0;
	string customer;
	int itemcodechoice;
	ITEM::node *sNode;//pointer as a node 
	float qty;
	
	
	//Show all items on the screen for bill entry
	yloc_add = sTmp.printitems();
	
	//Get the latest bill number and increment by 1
	billno = getlatestbillno(0);//will read the whole file for required bill no
	billno += 1;
	
	// Store bill with items in a temp.txt file. For every bill this will be overwritten
	tmpfile.open("temp.txt",ios::out);
	
	cout << "\n\tBill No.     :" << billno;
	cout << "\n\tCustomer Name:" ;
	cin >> customer;
	
	
	// Add header of the bill
	tmpfile << "\nBill No.:" << billno << endl;
	tmpfile << setfill(' ');
	tmpfile << setw(40) << "ABC Stores\n\n";
	tmpfile << "Customer:" << customer << endl;
	tmpfile << setfill('-') << setw(80) << " " << endl;
	tmpfile << setfill(' ');
	tmpfile << "Sr.|Code |Item Name           |Unit|     Price|  Quantity|         Amount|\n";	

	tmpfile << setfill('-') << setw(80) << " " << endl;
//	cout << "\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n\n" ;
	// Enter items
	yloc=yloc_add+14;
	gotoxy(0,yloc);
	yloc_add+=17;
	cout << "---------->Enter Item Code [0 to Exit]:" ;
	cin >> itemcodechoice;
	while(itemcodechoice != 0)
	{
		sNode = sTmp.search(itemcodechoice);
		if (sNode!=NULL)
		{
			gotoxy(39,yloc+1);
			cout << "                 "; // Clear line 	
			gotoxy(50,yloc);
			cout << "                             "; // Clear line 				
			gotoxy(0,yloc+1);
			cout << "------------------->Enter Quantity [1]:";
			
			cin >> qty;
			if (cin.fail())//if qty is negative or 0 then cin.fail activates 
			{
				cin.clear();//it clears the qty value
				qty = 1;//by default the qty becomes 1
			}

			if(firsttime==0)		
			{
					cout << "Sr.|Code |Item Name           |Unit|     Price|  Quantity|         Amount|\n";				
					firsttime=1;
			}		
			gotoxy(0,yloc_add);	
			//Item Details
			cout << setfill(' ');
			cout << right << setw(3) << srno+1;
			cout << "|";
			cout << left << setw(5) << sNode->itemcode;
			cout << "|";
			cout << setw(20) << sNode->itemname ;
			cout << "|";
			cout << setw(4) << sNode->unit ;
			cout << "|";
			cout << right << setw(10) << sNode->price;
			cout << "|";
			cout << setw(10) << qty;
			cout << "|";
			cout << setw(15) << qty*sNode->price;
			cout << "|";
			cout << "\n";			
			
			tmpfile << setfill(' ');
			tmpfile << right << setw(3) << srno+1;
			tmpfile << "|";
			tmpfile << left << setw(5) << sNode->itemcode;
			tmpfile << "|";
			tmpfile << setw(20) << sNode->itemname ;
			tmpfile << "|";
			tmpfile << setw(4) << sNode->unit ;
			tmpfile << "|";
			tmpfile << right << setw(10) << sNode->price;
			tmpfile << "|";
			tmpfile << setw(10) << qty;
			tmpfile << "|";
			tmpfile << setw(15) << qty*sNode->price;
			tmpfile << "|";
			tmpfile << "\n";


			totalamount = totalamount + qty*sNode->price ;
			
			gotoxy(40,yloc-3);	
			cout << "### Bill Amount: Rs." << totalamount << " ###";
			yloc_add += 1;
			srno+=1;
		}	
		else 
		{
			gotoxy(50,yloc);
			cout << "...Error:Item not available!!";
		}
		
		gotoxy(39,yloc);
		cout << "      "; // Clear line 
		gotoxy(0,yloc+1);
		cout << "                                                                     "; // Clear line 	
		gotoxy(0,yloc);	
		cout << "---------->Enter Item Code [0 to Exit]:" ;
	    cin >> itemcodechoice;
	}
	if (srno > 0)
	{
		tmpfile << setw(69) << "#########################################Bill Amount: Rs.|"  << totalamount <<"|"<< endl;
		
		//Footer
		tmpfile << setfill('-') << setw(80) << " " << endl;
		tmpfile << "Signature:\n";
		tmpfile.close();
		
		//update latest bill number in billno.txt
		getlatestbillno(billno);//the ongoing bill no

		
		// print and save bill
		system("CLS");
		cout<<"\n\nYour Bill:\n\n";
		
		// declaration
		fstream billfile;
		string line;

			
		billfile.open("bills.txt",ios::out| ios::app);
		tmpfile.open("temp.txt",ios::in);
			
		while(getline(tmpfile,line))
		{
			billfile << line << endl;//here line means whole sring with spaces 
			cout << line << endl;
		}
				
		tmpfile.close();
		billfile.close();
		cout<<"Press any key to continue...";
		getch();
	}
	else
	{
		tmpfile.close();	
	}
}


// Option is used to print the bill from bills.txt file. The bill number entered by the user is passed in this function.
// Function searches for the required bill number. If found it will start printing lines from bills.txt till the next
// bill number is reached.
// If bill number is not found then it will give proper message.
void printfile(int billno)
{
	fstream billfile;
	string line;
	int printline = 0;
	int isbillprinted = 0;
	
	billfile.open("bills.txt",ios::in);
	while (getline(billfile,line))
	{
		if (line.substr(0,9)== "Bill No.:")//this is a syntax to check whether "Bill No.:" is present from o to 0
		{
			if (printline==0)
			{
				if (atoi(line.substr(9).c_str())== billno)//bill no is taken as an integer and checks whether it is a requred bill
				{
					printline = 1;
					isbillprinted = 1;
					system("CLS");	
				}
			}
			else
			{
				printline = 0;
			}
		}
		
		if (printline==1)//whole bill is printed with one by one line 
		{
			cout << line <<endl;
		}
	}
	billfile.close();
	if (isbillprinted==0)
	{
		cout << "\nBill No.:" << billno << " not found!!" << endl;
	}
}


// Main Function
int main()
{
	ITEM s;
	char ch;
	int ino,noitems;
	
	//Read items from items.txt in a linked list.
	s.readitemfile();

	//Display Menu
	while ( 1)
	{
		system("CLS");	
		cout<<"\n******************[Billing System]******************\n";
		cout<<"\n                     ABC STORES                     \n";
		cout<<"\n\t\tMENU\n\n\t\t[B] Billing\n\n\t\t[P] Print Bill\n\n\t\t[L] List of Items\n\n\t\t[X] Exit\n";
		cout<<"\nEnter your choice [B/P/L/X]:";
		ch=getche();
	
		switch(ch)
		{
			case 'b':
			case 'B':
						billentry(s);
						break;
			case 'p':
			case 'P':
				//printin the items on the screen as well as in the temp file
						cout<<"\n\nEnter Bill No.:";
						cin>>ino;
						printfile(ino);
						cout<<"Press any key to continue...";
						getch();
						break;
			case 'l':
			case 'L':
						noitems = s.printitems(); 
						cout<<"Press any key to continue...";
						getch();
						break;
			case 'x':
			case 'X':
					exit(1);
					break;
			default:
					break;
		}
	}	
	return(0);
}
