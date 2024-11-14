# TASK-2-ATM-SIMULATION
// DIT-02-0058/2024
// Wambua Kelvin 

// TASK 2: ATM SIMULATION

#include <iostream>
#include <unordered_map>
#include <memory>
#include <string>

class Account {
private:
    int accountNumber;
    int pin;
    double balance;

public:
    Account(int accNumber, int pin, double initialBalance)
        : accountNumber(accNumber), pin(pin), balance(initialBalance) {}

    bool verifyPin(int inputPin) const {
        return pin == inputPin;
    }

    double getBalance() const {
        return balance;
    }

    void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    bool withdraw(double amount) {
        if (amount > 0 && balance >= amount) {
            balance -= amount;
            return true;
        }
        return false;
    }

    int getAccountNumber() const {
        return accountNumber;
    }
};

class Transaction {
protected:
    std::shared_ptr<Account> account;

public:
    Transaction(std::shared_ptr<Account> acc) : account(acc) {}

    virtual void execute() = 0;  // virtual function
    virtual ~Transaction() = default;
};

class Deposit : public Transaction {
private:
    double amount;

public:
    Deposit(std::shared_ptr<Account> acc, double amt) : Transaction(acc), amount(amt) {}

    void execute() override {
        account->deposit(amount);
        std::cout << "Deposited: " << amount << "\n";
    }
};

class Withdrawal : public Transaction {
private:
    double amount;

public:
    Withdrawal(std::shared_ptr<Account> acc, double amt) : Transaction(acc), amount(amt) {}

    void execute() override {
        if (account->withdraw(amount)) {
            std::cout << "Withdrew: " << amount << "\n";
        } else {
            std::cout << "No enough funds in your account.\n";
        }
    }
};

class ATM {
private:
    std::unordered_map<int, std::shared_ptr<Account>> accounts;

public:
    void addAccount(int accountNumber, int pin, double initialBalance) {
        accounts[accountNumber] = std::make_shared<Account>(accountNumber, pin, initialBalance);
    }

    std::shared_ptr<Account> authenticate(int accountNumber, int pin) {
        if (accounts.find(accountNumber) != accounts.end() && accounts[accountNumber]->verifyPin(pin)) {
            return accounts[accountNumber];
        }
        return nullptr;
    }

    void performTransaction(std::shared_ptr<Transaction> transaction) {
        transaction->execute();
    }
};

int main() {
    ATM atm;
    
    // account number included
    atm.addAccount(12345, 1234, 1000.0);

    int accountNumber, pin;
    std::cout << "Enter account number: ";
    std::cin >> accountNumber;
    std::cout << "Enter PIN: ";
    std::cin >> pin;

    std::shared_ptr<Account> account = atm.authenticate(accountNumber, pin);
    if (account) {
        std::cout << "Authenticated successfully.\n";

        int choice;
        do {
            std::cout << "\nSelect Transaction Type:\n1. Deposit\n2. Withdrawal\n3. Check Balance\n4. Cancel\nPlease enter a value : \n";
            std::cin >> choice;

            switch (choice) {
                case 1: {
                    double amount;
                    std::cout << "How much would you like to deposit: ";
                    std::cin >> amount;
                    std::shared_ptr<Transaction> deposit = std::make_shared<Deposit>(account, amount);
                    atm.performTransaction(deposit);
                    break;
                }
                case 2: {
                    double amount;
                    std::cout << "withdrawal amount: ";
                    std::cin >> amount;
                    std::shared_ptr<Transaction> withdrawal = std::make_shared<Withdrawal>(account, amount);
                    atm.performTransaction(withdrawal);
                    break;
                }
                case 3: {
                    std::cout << "Your Balance is: " << account->getBalance() << "\n";
                    break;
                }
                case 4:
                    std::cout << "Canceling...\n";
                    break;
                default:
                    std::cout << "...please enter a valid choice.\n";
            }
        } while (choice != 4);
    } else {
        std::cout << "Verification failed. \n";
    }

    return 0;
}
