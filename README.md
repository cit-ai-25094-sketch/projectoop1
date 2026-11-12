#include <iostream>
#include <vector>
#include <string>
#include <iomanip>
using namespace std;

// ---------- Abstract base class ----------
class MenuItem {
protected:
    int id;
    string name;
    double price;
public:
    MenuItem(int id, string name, double price)
        : id(id), name(name), price(price) {}
    virtual ~MenuItem() {}

    int getId() const { return id; }
    string getName() const { return name; }
    double getPrice() const { return price; }

    virtual string getType() const = 0;            // pure virtual
    virtual double getFinalPrice() const { return price; }

    friend ostream& operator<<(ostream& os, const MenuItem& m) {
        os << left << setw(4) << m.id << setw(18) << m.name
           << setw(10) << m.getType() << fixed << setprecision(2)
           << m.getFinalPrice() << " $";
        return os;
    }
};

// ---------- Derived classes ----------
class Food : public MenuItem {
public:
    Food(int id, string name, double price) : MenuItem(id, name, price) {}
    string getType() const override { return "Food"; }
};

class Drink : public MenuItem {
public:
    Drink(int id, string name, double price) : MenuItem(id, name, price) {}
    string getType() const override { return "Drink"; }
};

class Dessert : public MenuItem {
    double discount; // percentage
public:
    Dessert(int id, string name, double price, double discount = 10)
        : MenuItem(id, name, price), discount(discount) {}
    string getType() const override { return "Dessert"; }
    double getFinalPrice() const override { return price * (1 - discount / 100); }
};

// ---------- Order (composition) ----------
class Order {
    static int counter;
    int orderNo;
    int table;
    vector<MenuItem*> items;
    vector<int> qty;
public:
    Order(int table) : orderNo(++counter), table(table) {}

    void addItem(MenuItem* item, int q) {
        items.push_back(item);
        qty.push_back(q);
    }

    double total() const {
        double t = 0;
        for (size_t i = 0; i < items.size(); i++)
            t += items[i]->getFinalPrice() * qty[i];
        return t;
    }

    void printBill() const {
        cout << "\n======== Bill #" << orderNo << " | Table " << table << " ========\n";
        for (size_t i = 0; i < items.size(); i++)
            cout << left << setw(18) << items[i]->getName()
                 << "x" << setw(4) << qty[i] << fixed << setprecision(2)
                 << items[i]->getFinalPrice() * qty[i] << " $\n";
        cout << "---------------------------------\n";
        cout << "TOTAL: " << total() << " $\n";
    }
};
int Order::counter = 0;

// ---------- Restaurant ----------
class Restaurant {
    vector<MenuItem*> menu;
public:
    ~Restaurant() { for (auto m : menu) delete m; }

    void addItem(MenuItem* m) { menu.push_back(m); }

    void showMenu() const {
        cout << "\n----------- MENU -----------\n";
        for (auto m : menu) cout << *m << endl;
    }

    MenuItem* find(int id) const {
        for (auto m : menu) if (m->getId() == id) return m;
        return nullptr;
    }
};

// ---------- Main ----------
int main() {
    Restaurant r;
    r.addItem(new Food(1, "Burger", 6.5));
    r.addItem(new Food(2, "Pizza", 8.0));
    r.addItem(new Food(3, "Kebab", 9.5));
    r.addItem(new Drink(4, "Pepsi", 1.5));
    r.addItem(new Drink(5, "Tea", 1.0));
    r.addItem(new Dessert(6, "Cake", 4.0, 10));

    int choice;
    do {
        cout << "\n1) Show menu\n2) New order\n0) Exit\nChoice: ";
        cin >> choice;

        if (choice == 1) {
            r.showMenu();
        } else if (choice == 2) {
            int table;
            cout << "Table number: ";
            cin >> table;
            Order order(table);
            r.showMenu();

            int id, q;
            while (true) {
                cout << "Item id (0 to finish): ";
                cin >> id;
                if (id == 0) break;
                MenuItem* item = r.find(id);
                if (!item) { cout << "Item not found!\n"; continue; }
                cout << "Quantity: ";
                cin >> q;
                order.addItem(item, q);
            }
            order.printBill();
        }
    } while (choice != 0);

    cout << "Goodbye!\n";
    return 0;
}
