Inventory Manager is a high-performance desktop application, built with Qt (C++), designed for streamlined stock management and real-time profitability tracking.

Key Features:
Comprehensive Management: Rapidly add, edit, or remove inventory items.

Financial Insights: Automatic calculation of profit per unit and total potential profit per stock.

Sales Tracking: Built-in sales registration that updates stock quantities instantly.

Organized Workflow: Intuitive search functionality and one-click column sorting for fast data access.

Data Persistence: Automatic CSV data saving on exit and easy export functionality for external reporting (Excel/CSV compatible).

User-Friendly: Includes an "Undo" feature for quick recovery from accidental deletions.

This is a compact, portable, and efficient solution, perfect for anyone requiring a clear, structured, and organized way to track stock levels and monitor profit margins.

The test code was on VS CODE :


#include <iostream>
#include <string>
#include <vector>
#include <fstream>

using namespace std;

struct Part {
    string name;
    double buyPrice;
    double sellPrice;
    int quantity;

    double calculateProfit() const {
        return sellPrice - buyPrice;
    }
};

void saveInventory(const vector<Part>& inventory) {
    ofstream g("inventory.txt");
    
    if (!g.is_open()) {
        cout << "Error opening file for saving!" << endl;
        return;
    }

    for (const Part& p : inventory) {
        g << p.name << "\n";
        g << p.buyPrice << "\n";
        g << p.sellPrice << "\n";
        g << p.quantity << "\n";
    }

    g.close();
}

void loadInventory(vector<Part>& inventory) {
    ifstream f("inventory.txt");

    if (!f.is_open()) {
        return;
    }

    Part p;
    while (getline(f >> ws, p.name)) {
        f >> p.buyPrice;
        f >> p.sellPrice;
        f >> p.quantity;
        inventory.push_back(p);
    }

    f.close();
}

int main() {
    vector<Part> inventory;

    loadInventory(inventory);

    if (inventory.empty()) {
        Part p1 = {"Oil Filter", 40.0, 75.0, 5};
        Part p2 = {"Spark Plug", 20.0, 35.0, 10};
        Part p3 = {"Brake Pads", 15.0, 22.0, 23};
        
        inventory.push_back(p1);
        inventory.push_back(p2);
        inventory.push_back(p3);
        saveInventory(inventory);
    }

    int option = -1;

    while (option != 0) {
        cout << "\n=== INVENTORY MANAGEMENT MENU ===" << endl;
        cout << "1. Add new part" << endl;
        cout << "2. Display full inventory" << endl;
        cout << "3. Modify existing part" << endl;
        cout << "4. Delete part from inventory" << endl;
        cout << "5. Search part" << endl;
        cout << "6. Register a sale" << endl;
        cout << "0. Exit" << endl;
        cout << "Choose an option: ";
        cin >> option;

        switch (option) {
            case 1: {
                Part newPart;
                cout << "Enter part name: ";
                getline(cin >> ws, newPart.name);

                cout << "Enter buy price: ";
                cin >> newPart.buyPrice;

                cout << "Enter sell price: ";
                cin >> newPart.sellPrice;

                cout << "Enter quantity: ";
                cin >> newPart.quantity;

                inventory.push_back(newPart);
                saveInventory(inventory);

                cout << "-> Part added and saved successfully!" << endl;
                break;
            }
            case 2: {
                double totalProfit = 0.0;
                cout << "\n--- CURRENT INVENTORY ---" << endl;

                for (const Part& p : inventory) {
                    cout << "Part: " << p.name << " | Quantity: " << p.quantity << endl;
                    cout << "Buy price: $" << p.buyPrice << " | Sell price: $" << p.sellPrice << endl;
                    cout << "Unit profit: $" << p.calculateProfit() << endl;
                    cout << "Total profit on stock: $" << p.calculateProfit() * p.quantity << endl;
                    cout << "----------------------------------" << endl;
                    totalProfit += p.calculateProfit() * p.quantity;
                }

                cout << "==================================" << endl;
                cout << "Total potential profit: $" << totalProfit << endl;
                cout << "==================================" << endl;
                break;
            }
            case 3: {
                string searchName;
                cout << "\nEnter the name of the part you want to modify: ";
                getline(cin >> ws, searchName);

                bool found = false;

                for (Part &p : inventory) {
                    if (p.name == searchName) {
                        found = true;
                        cout << "\n--- Part found! ---" << endl;

                        cout << "New buy price (current: " << p.buyPrice << "): ";
                        cin >> p.buyPrice;

                        cout << "New sell price (current: " << p.sellPrice << "): ";
                        cin >> p.sellPrice;

                        cout << "New quantity (current: " << p.quantity << "): ";
                        cin >> p.quantity;

                        saveInventory(inventory);
                        cout << "-> Part updated and saved successfully!" << endl;
                        break;
                    }
                }

                if (!found) {
                    cout << "-> Error: Part '" << searchName << "' was not found in inventory!" << endl;
                }

                break;
            }
            case 4: {
                string searchName;
                cout << "\nEnter the name of the part you want to delete: ";
                getline(cin >> ws, searchName);

                bool found = false;

                for (auto it = inventory.begin(); it != inventory.end(); ++it) {
                    if (it->name == searchName) {
                        inventory.erase(it);
                        found = true;
                        saveInventory(inventory);
                        cout << "-> Part deleted successfully!" << endl;
                        break;
                    }
                }

                if (!found) {
                    cout << "-> Error: Part '" << searchName << "' was not found!" << endl;
                }

                break;
            }
            case 5: {
                string query;
                cout << "\nEnter part name or keyword: ";
                getline(cin >> ws, query);

                bool found = false;
                cout << "\n--- SEARCH RESULTS ---" << endl;

                for (const Part& p : inventory) {
                    if (p.name.find(query) != string::npos) {
                        cout << "Part: " << p.name << " | Stock: " << p.quantity 
                             << " | Price: $" << p.sellPrice << endl;
                        found = true;
                    }
                }

                if (!found) {
                    cout << "No parts matched your search query." << endl;
                }

                break;
            }
            case 6: {
                string searchName;
                cout << "\nEnter sold part name: ";
                getline(cin >> ws, searchName);

                bool found = false;

                for (Part &p : inventory) {
                    if (p.name == searchName) {
                        found = true;
                        int soldQuantity;
                        cout << "Current stock for " << p.name << ": " << p.quantity << endl;
                        cout << "How many units were sold? ";
                        cin >> soldQuantity;

                        if (soldQuantity > p.quantity) {
                            cout << "-> Error: Not enough units in stock!" << endl;
                        } else {
                            p.quantity -= soldQuantity;
                            double totalRevenue = soldQuantity * p.sellPrice;
                            double saleProfit = soldQuantity * p.calculateProfit();

                            saveInventory(inventory);
                            cout << "-> Sale registered! Total revenue: $" << totalRevenue 
                                 << " | Profit from sale: $" << saleProfit << endl;
                        }
                        break;
                    }
                }

                if (!found) {
                    cout << "-> Error: Part '" << searchName << "' does not exist in inventory!" << endl;
                }

                break;
            }
            case 0:
                saveInventory(inventory);
                cout << "Data saved successfully. Goodbye!" << endl;
                break;
            default:
                cout << "Invalid option! Please try again." << endl;
        }
    }

    return 0;
}
