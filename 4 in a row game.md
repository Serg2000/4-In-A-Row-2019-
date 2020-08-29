# 4-In-A-Row-2019-
#include <iostream>
#include <vector>
#include <sstream>
#include <fstream>
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <time.h>

using namespace std;

string print(const vector<vector<char>>&, int); // prints the 2D vector and highlights the wining row
void print_into_file(int, string); // prints the final value in a new created file sorted in increasing order
void change_table_size(vector<vector<char>>&, vector<int>&); // Asks the user row and column size for the table
void clear(vector<vector<char>>&, vector<int>&); // Clears the 2D array assigning all elements to 'O'
void dropping(vector<vector<char>>&, vector<int>&, int, char); // Everytime this function is called, it drops the ball down ? Takes the 2D vector & column number

bool checker(const vector<vector<char>>&, int, int, int, string); // Checker is created by 3 functions that check 3 ways
int horizontal_checker(const vector<vector<char>>&, int, int, int); //Checks horizontally '-'
int vertical_checker(const vector<vector<char>>&, int, int, int); // Checks vertically '|'
int diagonal_checker1(const vector<vector<char>>&, int, int, int); // Checks diagonally '/'
int diagonal_checker2(const vector<vector<char>>&, int, int, int); // Checks diagonally '\'

void save_to_txt_file(const vector<vector<char>>&, const vector<int>&, string, string, string, int, int, int);
bool open_saved_txt_file(vector<vector<char>>&, vector<int>&, string, string&, string&, int&, int&, int&); //returns true if the file exists

int computer(vector<vector<char>>&, vector<int>& tracker); // Randomly generates column number in ranges of columns
int checker_for_AI(const vector<vector<char>>& game, int row, int column);

int main()
{
	srand(time(NULL));
	int row = 0, column = 0;
	string input;
	string filename;
	bool flag, computer_flag;
	string step_recorder;
	string red_user, blue_user;      // Asks players' names

	int game_counter = 0;
	int first_score = 0; 
	int second_score = 0;

	vector < vector<char>> game(6, vector<char>(7)); // Default table size is 6 X 7, to change the size user should enter change
	vector<int> tracker(7); //Tracks in which row the inputed element is. 
	while (true)
	{
		computer_flag = false;
		cout << "To play against computer, enter computer. \nTo play with 2 players, enter two. ";
		cin >> input;
		cout << endl;

		if (input == "computer")
		{
			cout << "What is your name: ";
			cin >> red_user;
			blue_user = "Computer";
			computer_flag = true;
			cout << endl;
			break;
		}
		else if (input == "two")
		{
			cout << "What is the first player's name: ";
			cin >> red_user;
			cout << "What is the second player's name: ";
			cin >> blue_user;
			cout << endl;
			break;
		}
		cout << endl;
	}
	while (true)
	{
		flag = false;
		int counter = 0;
		clear(game, tracker);
		string input;
		cout << "\nTo quit the game, enter q or quit anytime. \nTo start a new game, enter y or yes.";
		cout << "\nTo save the game, enter s or save anytime. \nTo continue a saved game, enter continue."; 
		cout << "\nDefault size of the table is set to 6 X 7. If you want to change, enter change."; 
		cout << "\nINPUT: ";
		cin >> input;

		if (input == "y" || input == "yes") // new game, asks names and if the user wants to change the size of the table
		{	
			flag = true;			
			print(game, counter);
		}
		else if (input == "q" || input == "quit")
		{
			break;
		}
		else if (input == "s" || input == "save")
		{
			cout << endl << "Enter the file's name: ";
			cin >> filename;
			save_to_txt_file(game, tracker, filename, red_user, blue_user, first_score, second_score, counter);
		}
		else if (input == "continue")
		{
			game.resize(0);
			cout << "Enter the file's name you want to open: ";
			checkfilename:
			cin >> filename;
			// Checks if the file exists and if it does, it assigns the content of the text file to those variables 
			flag = open_saved_txt_file(game, tracker, filename, red_user, blue_user, first_score, second_score, counter);
			if (!flag)
			{
				cout << "\nThe entered file does not exist. Try another one: ";
				goto checkfilename;
			}
			else if (filename == "q" || filename == "quit")
				continue;
			else
				print(game, counter);
		}
		else if (input == "change") 
		{
			change_table_size(game, tracker);
			cout << endl << "You have successfully changed the size of the table. " << endl;
		}

		string turn;
		char symbol;
		while (flag)
		{
			counter++;
			int column_number = 0;
			turn = counter % 2 == 1 ? red_user : blue_user;
			cout << "\t\t\t\t\t\t" << turn << "'s turn. Column number: ";

		if(computer_flag && counter % 2 == 0)
		{
			symbol = 'B';

			column_number = computer(game, tracker);
			while(tracker[column_number] <= 0)
				column_number = rand() % tracker.size();
			cout << column_number + 1 << endl;
		}

		else { 
			 //Checks whose turn 
			
			inputcheck: //Instead of while loop
				cin >> input;

				if (input == "s" || input == "save")
				{
					cout << endl << "Enter the file's name: ";
					cin >> filename;
					counter--;
					save_to_txt_file(game, tracker, filename, red_user, blue_user, first_score, second_score, counter);
					break;
				}
				else if (input == "q" || input == "quit")
				{
					save_to_txt_file(game, tracker, "last_game", red_user, blue_user, first_score, second_score, counter);
					break;
				}
				column_number = stoi(input);
				if (column_number > game[0].size() || column_number < 1) //Checks the range of input to match column number
				{
					cout << endl << "The input is out of size. Please, enter a valid column number. ";
					goto inputcheck;
				}

				column_number--; // All column_numbers have to be presented as substructed by one for passing to the function as indexes
				if (counter % 2 == 1)
				{
					symbol = 'R';
				}
				else
				{
					symbol = 'B';
				}
			}
			
			if (tracker[column_number] > 0)
			{
				dropping(game, tracker, column_number, symbol);
				SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 14);
				flag = checker(game, tracker[column_number], column_number, counter, turn); //returns false if someone won
				SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 15);
				if (!flag)
				{
					game_counter++;
					if (counter % 2 == 1)
					{
						first_score++;
					}
					else
					{
						second_score++;
					}
					cout << endl << "\t\t\t\t\t\t" << red_user << ": " << first_score << "   " << blue_user << ": " << second_score << endl;
				}
				step_recorder += print(game, counter); //it's the sum of all played boards
			}
			else
			{
				
				cout << endl << "This column is full. Please choose another one. " << endl;
				counter--;
			}
				
		}
		print_into_file(game_counter, step_recorder);
	}
	return 0;
}

// Table related functions
void change_table_size(vector<vector<char>>& game, vector<int>& tracker)
{
	int row, column;
	cout << "How many rows? ";
	cin >> row;
	cout << "How many columns? ";
	cin >> column;
	cout << endl;
	game.clear();
	game.resize(row);
	for (vector<char>& val : game)
		val.resize(column);
	tracker.resize(column);
	clear(game, tracker);
}
string print(const vector < vector<char>>& game, int turn)
{
	string output = "";
	for (int i = 0; i < game.size(); i++)
	{
		cout << endl << "\t\t\t\t\t\t";
		output = output + "\n" + "\t\t\t\t\t\t";
		for (int j = 0; j < game[i].size(); j++)
		{
			if (game[i][j] == 'R')
			{
				SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 4);
				cout << "R ";
				SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 15);
			}
			else if (game[i][j] == 'B')
			{
				SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 11);
				cout << "B ";
				SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), 15);
			}
			else
			{
				cout << game[i][j] << " ";
			}
			output = output + game[i][j] + " ";
		}
	}
	output += "\n\n";
	cout << endl << endl;
	return output;
}
void print_into_file(int game_counter, string output)
{
	string filename = "game" + to_string(game_counter) + ".txt";
	ofstream outfile;
	outfile.open(filename.c_str());
	outfile << output;
	outfile.close();
}
void clear(vector<vector<char>>& game, vector<int>& tracker)
{
	for (int i = 0; i < game.size(); i++)
	{
		for (int j = 0; j < game[0].size(); j++)
		{
			game[i][j] = 'O';
			tracker[j] = game.size() - 1; 		 // All trackers are initialized to the last row index number
		}
	}
}
//Storing and openning functions
void save_to_txt_file(const vector<vector<char>>& game, const vector<int>& tracker, string filename, string first_player_name,
	string second_player_name, int first_player_score, int second_player_score, int counter)
{
	ofstream save;
	filename += ".txt";
	save.open(filename.c_str());
	for (int i = 0; i < game.size(); i++)
	{
		for (int j = 0; j < game[i].size(); j++)
		{
			save << game[i][j] << " ";
		}
		save << endl;
	}
	save << endl;
	for (int val : tracker)
		save << val << " ";
	save << endl << endl;
	save << first_player_name << endl << endl << second_player_name << endl << endl;
	save << first_player_score << endl << endl << second_player_score << endl << endl << counter << endl << endl;
	save.close();
	cout << endl << "The game has been successfully saved in " << filename << " file. " << endl;
}
bool open_saved_txt_file(vector<vector<char>>& game, vector<int>& tracker, string filename, string& first_player_name,
	string& second_player_name, int& first_player_score, int& second_player_score, int& turn)
{
	filename += ".txt";
	ifstream infile;
	infile.open(filename.c_str());
	int counter = 0;
	int i = 0;
	string input;
	string sum = "";
	bool result;
	if (infile)
	{
		while (getline(infile, input))
		{
			if (input.size() == 0)
			{
				counter++;
				continue;
			}
			if (counter < 1)
			{
				game.push_back(vector<char>(0));
				for (int j = 0; j < input.size(); j++)
				{
					if (input[j] == 32)
						continue;
					game[i].push_back(input[j]);
				}
				i++;
			}
			else if (counter < 2)
			{
				i = 0;
				tracker.resize(game[0].size());
				for (int j = 0; j < input.size(); j++)
				{
					if (input[j] == 32)
					{
						sum.clear();
						continue;
					}
					sum += input[j];
					tracker[i] = stoi(sum);
					i++;
				}
			}
			else if (counter < 3)
			{
				first_player_name = input;
			}
			else if (counter < 4)
			{
				second_player_name = input;
			}
			else if (counter < 5)
			{
				first_player_score = stoi(input);
			}
			else if (counter < 6)
			{
				second_player_score = stoi(input);
			}
			else if (counter < 7)
			{
				turn = stoi(input);
			}
		}
		result = true;
	}
	else
	{
		result = false;
	}
	infile.close();
	return result;
}
//Checker function
bool checker(const vector<vector<char>>& game, int row, int column, int quant, string turn)
{
	const int counter = 0;
	int temp; // Receives the return value of counter from functions

	// Horizontal
	temp = horizontal_checker(game, row, column, counter);
	if (temp == 4)
	{
		cout << endl << "\t\t\t\t\t\t" << turn << " won! " << endl;
		return false;
	}

	// Vertical
	if (row < game.size() - 3)
	{
		temp = vertical_checker(game, row, column, 1 + counter);
		if (temp == 4)
		{
			cout << endl << "\t\t\t\t\t\t" << turn << " won! " << endl;
			return false;
		}
	}

	// Diagonal checker 1 '/'
	temp = diagonal_checker1(game, row, column, counter);
	if (temp == 4)
	{
		cout << endl << "\t\t\t\t\t\t" << turn << " won! " << endl;
		return false;
	}
	// Diagonal checker 2 '\'
	temp = diagonal_checker2(game, row, column, counter);
	if (temp == 4)
	{
		cout << endl << "\t\t\t\t\t\t" << turn << " won! " << endl;
		return false;
	}

	// Checks if the field is full
	if (quant == game.size() * game[0].size())
	{
		cout << endl << "\t\t\t\t\t\t" << "It's Draw!" << endl;
		return false;
	}

	return true; // Otherwise return true to continue the game
}
int diagonal_checker2(const vector<vector<char>>& game, int row, int column, int counter)
{
	int row_size = game.size();
	int col_size = game[0].size();
	char temp = game[row][column];
	for (int i = row - 3; i <= row + 3; i++)
	{
		if ((counter < 4) && (i < row_size) && (i > -1) && (column + i - row < col_size) && (column + i - row > -1)) //Border check
		{
			if (game[i][column + i - row] == temp)
				counter++;
			else
				counter = 0;
		}
	}
	return counter;
}
int diagonal_checker1(const vector<vector<char>>& game, int row, int column, int counter)
{
	int row_size = game.size();
	int col_size = game[0].size();
	char temp = game[row][column];
	for (int i = row + 3; i >= row - 3; i--)
	{
		if ((counter < 4) && (i < row_size) && (i > -1) && (column - i + row < col_size) && (column - i + row > -1)) //Border check
		{
			if (game[i][column - i + row] == temp)
				counter++;
			else
				counter = 0;
		}
	}
	return counter;
}
int horizontal_checker(const vector<vector<char>>& game, int row, int column, int counter)
{
	char temp = game[row][column];
	for (int i = column - 3; i <= column + 3; i++)
	{
		if ((counter < 4) && (i > -1) && (i < game[0].size())) //Border check
		{
			if (game[row][i] == temp)
				counter++;
			else
				counter = 0;
		}
	}

	return counter;
}
int vertical_checker(const vector<vector<char>>& game, int row, int column, int counter)
{
	if (counter < 4)
	{
		if (game[row][column] == game[row + 1][column])
			return vertical_checker(game, row + 1, column, counter + 1);
	}
	return counter; // Returns adding one to count entered element too
}
void dropping(vector<vector<char>>& game, vector<int>& row_position, int column_index_number, char character)
{
	/* row_position is a tracker */
	// Every time i is used as static variable taken from updated tracker vector
	for (int i = row_position[column_index_number]; i >= 0; i--)
	{
		if (game[i][column_index_number] == 'O')
		{
			game[i][column_index_number] = character;
			row_position[column_index_number] = i;
			break;
		}
	}
	
}
//AI related funcitons
int computer(vector<vector<char>>& game, vector<int>& tracker)
{
	vector<vector<char>> testing(game);
	vector<int> temp_tracker(tracker);

	int max = 1;
	int R_temp;
	int B_temp;
	int index = rand() % tracker.size();
	int R_index = -1;
	for (int i = 0; i < game[0].size(); i++)
	{
		dropping(testing, temp_tracker, i, 'B');
		B_temp = checker_for_AI(testing, temp_tracker[i], i);

		if (B_temp != 4)
		{
			dropping(testing, temp_tracker, i, 'R');
			R_temp = checker_for_AI(testing, temp_tracker[i], i);
			if (R_temp == 4)
			{
				testing[temp_tracker[i]][i] = 'O';
				testing[temp_tracker[i] + 1][i] = 'O';
				temp_tracker[i] = tracker[i];
				R_index = i;
			}
			else
			{
				testing[temp_tracker[i]][i] = 'O';
				testing[temp_tracker[i] + 1][i] = 'O';
				temp_tracker[i] = tracker[i];
				dropping(testing, temp_tracker, i, 'R');
				R_temp = checker_for_AI(testing, temp_tracker[i], i);
				if (R_temp == 4)
				{
					index = i;
					break;
				}
				else if (R_index != i && B_temp > max)
				{
					max = B_temp;
					index = i;
				}
				testing[temp_tracker[i]][i] = 'O';
				temp_tracker[i] = tracker[i];
			}
		}
		else
		{
			index = i;
			break;
		}
	}
	return index;
}
int checker_for_AI(const vector<vector<char>>& game, int row, int column)
{
	const int counter = 0;
	int max, temp; // Receives the return value of counter from functions
	max = 1;
	// Horizontal
	temp = horizontal_checker(game, row, column, counter);
	if (temp > max)
	{
		max = temp;
	}
	// Vertical
	if (row < game.size() - 3)
	{
		temp = vertical_checker(game, row, column, 1 + counter);
		if (temp > max)
		{
			max = temp;
		}
	}

	// Diagonal checker 1 '/'
	temp = diagonal_checker1(game, row, column, counter);
	if (temp > max)
	{
		max = temp;
	}
	// Diagonal checker 2 '\'
	temp = diagonal_checker2(game, row, column, counter);
	if (temp > max)
	{
		max = temp;
	}
	return max;
}
