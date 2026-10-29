import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.Scanner;

class User {
    private String username;
    private String email;
    private String password;

    public User(String username, String email, String password) {
        this.username = username;
        this.email = email;
        this.password = password;
    }

    public boolean login(String inputEmail, String inputPassword) {
        return email.equals(inputEmail) && password.equals(inputPassword);
    }

    public String getUsername() {
        return username;
    }

    public String getEmail() {
        return email;
    }
}

class Bill {
    private String name;
    private double amount;
    private Date dueDate;
    private String category;

    public Bill(String name, double amount, Date dueDate, String category) {
        this.name = name;
        this.amount = amount;
        this.dueDate = dueDate;
        this.category = category;
    }

    public boolean isOverdue() {
        return new Date().after(dueDate);
    }

    public String getName() { return name; }
    public double getAmount() { return amount; }
    public String getCategory() { return category; }
}

abstract class Account {
    protected int accountNumber;
    protected double balance;
    protected List<String> transactions;

    public Account(int accountNumber) {
        this.accountNumber = accountNumber;
        this.balance = 0.0;
        this.transactions = new ArrayList<>();
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            transactions.add("Deposit: +PHP " + amount);
            System.out.println("Successfully deposited PHP " + amount);
        } else {
            System.out.println("Invalid amount!");
        }
    }

    public boolean withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            transactions.add("Withdraw: -PHP " + amount);
            System.out.println("Successfully withdrew PHP " + amount);
            return true;
        }
        System.out.println("Insufficient balance or invalid amount!");
        return false;
    }

    public double getBalance() {
        return balance;
    }

    public List<String> getTransactions() {
        return transactions;
    }

    public abstract String getAccountSummary();
}

class PersonalAccount extends Account {
    private double savingsGoal;

    public PersonalAccount(int accountNumber) {
        super(accountNumber);
        this.savingsGoal = 0.0;
    }

    public void setSavingsGoal(double goal) {
        this.savingsGoal = goal;
        System.out.println("Savings goal set to PHP " + goal);
    }

    public double checkSavings() {
        if (savingsGoal <= 0) return 0;
        return (balance / savingsGoal) * 100;
    }

    public String getAccountSummary() {
        return "\nPERSONAL ACCOUNT SUMMARY\n" +
               "Account No: " + accountNumber + "\n" +
               "Balance: PHP " + String.format("%.2f", balance) + "\n" +
               "Savings Goal Progress: " + String.format("%.1f", checkSavings()) + "%\n";
    }
}

class SharedAccount extends Account {
    private List<User> members;
    private List<Bill> bills;

    public SharedAccount(int accountNumber) {
        super(accountNumber);
        this.members = new ArrayList<>();
        this.bills = new ArrayList<>();
    }

    public void addMember(User user) {
        if (!members.contains(user)) {
            members.add(user);
            System.out.println(user.getUsername() + " added to shared account");
        }
    }

    public void removeMember(User user) {
        members.remove(user);
        System.out.println(user.getUsername() + " removed from shared account");
    }

    public void addBill(Bill bill) {
        bills.add(bill);
        System.out.println("Bill added: " + bill.getName() + " - PHP " + bill.getAmount());
    }

    public double splitExpense(double totalAmount) {
        if (members.isEmpty()) return totalAmount;
        double share = totalAmount / members.size();
        System.out.println("Each member pays: PHP " + String.format("%.2f", share));
        return share;
    }

    public String getAccountSummary() {
        StringBuilder sb = new StringBuilder();
        sb.append("\nSHARED HOUSEHOLD ACCOUNT\n");
        sb.append("Account No: ").append(accountNumber).append("\n");
        sb.append("Total Balance: PHP ").append(String.format("%.2f", balance)).append("\n");
        sb.append("Members: ").append(members.size()).append("\n");
        for (User m : members) sb.append("  - ").append(m.getUsername()).append("\n");
        sb.append("Bills: ").append(bills.size()).append("\n");
        for (Bill b : bills) sb.append("  - ").append(b.getName()).append(": PHP ").append(b.getAmount()).append(" (").append(b.getCategory()).append(")\n");
        return sb.toString();
    }
}

public class FinanceSystem {
    private static Scanner sc = new Scanner(System.in);
    private static User currentUser;
    private static PersonalAccount personalAccount;
    private static SharedAccount sharedAccount;

    public static void main(String[] args) {
        System.out.println("PERSONAL FINANCE PARTNER");
        mainMenu();
    }

    private static void mainMenu() {
        while (true) {
            System.out.println("\nMAIN MENU");
            System.out.println("1. Sign Up");
            System.out.println("2. Log In");
            System.out.println("3. Exit");
            System.out.print("Enter choice: ");
            
            if (!sc.hasNextInt()) { sc.next(); continue; }
            int choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1: signUp(); break;
                case 2: login(); break;
                case 3: System.out.println("Goodbye!"); return;
                default: System.out.println("Invalid choice!");
            }
        }
    }

    private static void signUp() {
        System.out.print("Enter username: ");
        String user = sc.nextLine();
        System.out.print("Enter email: ");
        String email = sc.nextLine();
        System.out.print("Enter password: ");
        String pass = sc.nextLine();
        currentUser = new User(user, email, pass);
        personalAccount = new PersonalAccount(1001);
        sharedAccount = new SharedAccount(2001);
        System.out.println("Account created successfully! Please log in.");
    }

    private static void login() {
        System.out.print("Enter email: ");
        String email = sc.nextLine();
        System.out.print("Enter password: ");
        String pass = sc.nextLine();
        
        if (currentUser != null && currentUser.login(email, pass)) {
            System.out.println("Welcome, " + currentUser.getUsername() + "!");
            financeMainMenu();
        } else {
            System.out.println("Account not found. Please sign up first.");
        }
    }

    private static void financeMainMenu() {
        while (true) {
            System.out.println("\nFINANCE MAIN MENU");
            System.out.println("Username: " + currentUser.getUsername());
            System.out.println("Your Balance: PHP " + String.format("%.2f", personalAccount.getBalance()));
            System.out.println("\n1. Individual Wallet");
            System.out.println("2. Household Shared Account");
            System.out.println("3. Log Out");
            System.out.print("Enter choice: ");
            
            if (!sc.hasNextInt()) { sc.next(); continue; }
            int choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1: personalWalletMenu(); break;
                case 2: sharedAccountMenu(); break;
                case 3: System.out.println("Logged out."); return;
                default: System.out.println("Invalid choice!");
            }
        }
    }

    private static void personalWalletMenu() {
        while (true) {
            System.out.println("\nINDIVIDUAL WALLET");
            System.out.println("Balance: PHP " + String.format("%.2f", personalAccount.getBalance()));
            System.out.println("1. Deposit");
            System.out.println("2. Withdraw");
            System.out.println("3. View Summary");
            System.out.println("4. Set Savings Goal");
            System.out.println("5. Exit");
            System.out.print("Enter choice: ");
            
            if (!sc.hasNextInt()) { sc.next(); continue; }
            int choice = sc.nextInt();

            switch (choice) {
                case 1:
                    System.out.print("Enter amount to deposit: ");
                    double dep = sc.nextDouble();
                    personalAccount.deposit(dep);
                    break;
                case 2:
                    System.out.print("Enter amount to withdraw: ");
                    double with = sc.nextDouble();
                    personalAccount.withdraw(with);
                    break;
                case 3:
                    System.out.println(personalAccount.getAccountSummary());
                    break;
                case 4:
                    System.out.print("Enter your savings goal: ");
                    double goal = sc.nextDouble();
                    personalAccount.setSavingsGoal(goal);
                    break;
                case 5: return;
                default: System.out.println("Invalid choice!");
            }
        }
    }

    private static void sharedAccountMenu() {
        while (true) {
            System.out.println("\nHOUSEHOLD ACCOUNT");
            System.out.println(sharedAccount.getAccountSummary());
            System.out.println("\n1. Add Member");
            System.out.println("2. Add Bill");
            System.out.println("3. Split Expense");
            System.out.println("4. Deposit to Shared Account");
            System.out.println("5. Exit");
            System.out.print("Enter choice: ");
            
            if (!sc.hasNextInt()) { sc.next(); continue; }
            int choice = sc.nextInt();
            sc.nextLine();

            switch (choice) {
                case 1:
                    System.out.print("Enter member name: ");
                    String name = sc.nextLine();
                    sharedAccount.addMember(new User(name, name + "@email.com", "1234"));
                    break;
                case 2:
                    System.out.print("Bill name: ");
                    String bName = sc.nextLine();
                    System.out.print("Amount: ");
                    double amt = sc.nextDouble();
                    sc.nextLine();
                    System.out.print("Category (Needs/Wants/Savings): ");
                    String cat = sc.nextLine();
                    sharedAccount.addBill(new Bill(bName, amt, new Date(), cat));
                    break;
                case 3:
                    System.out.print("Total expense amount: ");
                    double total = sc.nextDouble();
                    sharedAccount.splitExpense(total);
                    break;
                case 4:
                    System.out.print("Amount to deposit: ");
                    double dep = sc.nextDouble();
                    sharedAccount.deposit(dep);
                    break;
                case 5: return;
                default: System.out.println("Invalid choice!");
            }
        }
    }
}
