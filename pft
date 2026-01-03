import java.io.*;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.*;
import java.util.stream.Collectors;

public class PersonalFinanceTracker {
    // Data models
    public static class Transaction {
        private int id;
        private String description;
        private double amount;
        private String category;
        private Date date;
        private String type; // "Income" or "Expense"
       
        public Transaction(int id, String description, double amount, String category, Date date, String type) {
            this.id = id;
            this.description = description;
            this.amount = amount;
            this.category = category;
            this.date = date;
            this.type = type;
        }

        // Getters and setters
        public int getId() { return id; }
        public String getDescription() { return description; }
        public double getAmount() { return amount; }
        public String getCategory() { return category; }
        public Date getDate() { return date; }
        public String getType() { return type; }
       
        @Override
        public String toString() {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            return String.format("%d | %s | %s | %.2f | %s | %s",
                id, sdf.format(date), description, amount, category, type);
        }
    }

    public static class Account {
        private int id;
        private String name;
        private double balance;
        private List<Transaction> transactions;
       
        public Account(int id, String name, double initialBalance) {
            this.id = id;
            this.name = name;
            this.balance = initialBalance;
            this.transactions = new ArrayList<>();
        }
       
        public void addTransaction(Transaction transaction) {
            transactions.add(transaction);
            if (transaction.getType().equalsIgnoreCase("Income")) {
                balance += transaction.getAmount();
            } else {
                balance -= transaction.getAmount();
            }
        }
       
        // Getters
        public int getId() { return id; }
        public String getName() { return name; }
        public double getBalance() { return balance; }
        public List<Transaction> getTransactions() { return transactions; }
    }

    public static class Budget {
        private String category;
        private double limit;
        private double currentSpending;
       
        public Budget(String category, double limit) {
            this.category = category;
            this.limit = limit;
            this.currentSpending = 0;
        }
       
        public void addSpending(double amount) {
            currentSpending += amount;
        }
       
        // Getters
        public String getCategory() { return category; }
        public double getLimit() { return limit; }
        public double getCurrentSpending() { return currentSpending; }
        public double getRemaining() { return limit - currentSpending; }
    }

    // Data storage handler
    public static class FileDataHandler {
        private static final String TRANSACTIONS_FILE = "transactions.csv";
        private static final String ACCOUNTS_FILE = "accounts.csv";
        private static final String BUDGETS_FILE = "budgets.csv";
        private static final SimpleDateFormat DATE_FORMAT = new SimpleDateFormat("yyyy-MM-dd");
       
        public void saveAllData(List<Account> accounts, List<Budget> budgets) {
            saveTransactions(accounts.stream()
                .flatMap(acc -> acc.getTransactions().stream())
                .collect(Collectors.toList()));
            saveAccounts(accounts);
            saveBudgets(budgets);
        }
       
        public List<Transaction> loadTransactions() {
            List<Transaction> transactions = new ArrayList<>();
            try (BufferedReader reader = new BufferedReader(new FileReader(TRANSACTIONS_FILE))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    String[] parts = line.split(",");
                    if (parts.length == 6) {
                        int id = Integer.parseInt(parts[0]);
                        Date date = DATE_FORMAT.parse(parts[1]);
                        String desc = parts[2];
                        double amount = Double.parseDouble(parts[3]);
                        String category = parts[4];
                        String type = parts[5];
                        transactions.add(new Transaction(id, desc, amount, category, date, type));
                    }
                }
            } catch (IOException | ParseException e) {
                // File doesn't exist or is empty - return empty list
            }
            return transactions;
        }
       
        public List<Account> loadAccounts(List<Transaction> allTransactions) {
            List<Account> accounts = new ArrayList<>();
            try (BufferedReader reader = new BufferedReader(new FileReader(ACCOUNTS_FILE))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    String[] parts = line.split(",");
                    if (parts.length >= 3) {
                        int id = Integer.parseInt(parts[0]);
                        String name = parts[1];
                        double balance = Double.parseDouble(parts[2]);
                        Account acc = new Account(id, name, balance);
                       
                        // Add transactions to account (if any)
                        if (parts.length > 3) {
                            String[] transIds = parts[3].split(";");
                            for (String transId : transIds) {
                                if (!transId.isEmpty()) {
                                    int tid = Integer.parseInt(transId);
                                    allTransactions.stream()
                                        .filter(t -> t.getId() == tid)
                                        .findFirst()
                                        .ifPresent(acc::addTransaction);
                                }
                            }
                        }
                        accounts.add(acc);
                    }
                }
            } catch (IOException e) {
                // Create default account if file doesn't exist
                Account defaultAccount = new Account(1, "Default Account", 0);
                accounts.add(defaultAccount);
            }
            return accounts;
        }
       
        public List<Budget> loadBudgets() {
            List<Budget> budgets = new ArrayList<>();
            try (BufferedReader reader = new BufferedReader(new FileReader(BUDGETS_FILE))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    String[] parts = line.split(",");
                    if (parts.length == 3) {
                        String category = parts[0];
                        double limit = Double.parseDouble(parts[1]);
                        double spending = Double.parseDouble(parts[2]);
                        Budget budget = new Budget(category, limit);
                        budget.addSpending(spending);
                        budgets.add(budget);
                    }
                }
            } catch (IOException e) {
                // Return empty list if file doesn't exist
            }
            return budgets;
        }
       
        private void saveTransactions(List<Transaction> transactions) {
            try (PrintWriter writer = new PrintWriter(new FileWriter(TRANSACTIONS_FILE))) {
                for (Transaction t : transactions) {
                    writer.println(String.format("%d,%s,%s,%.2f,%s,%s",
                        t.getId(),
                        DATE_FORMAT.format(t.getDate()),
                        t.getDescription(),
                        t.getAmount(),
                        t.getCategory(),
                        t.getType()));
                }
            } catch (IOException e) {
                System.err.println("Error saving transactions: " + e.getMessage());
            }
        }
       
        private void saveAccounts(List<Account> accounts) {
            try (PrintWriter writer = new PrintWriter(new FileWriter(ACCOUNTS_FILE))) {
                for (Account acc : accounts) {
                    String transIds = acc.getTransactions().stream()
                        .map(t -> String.valueOf(t.getId()))
                        .collect(Collectors.joining(";"));
                    writer.println(String.format("%d,%s,%.2f,%s",
                        acc.getId(),
                        acc.getName(),
                        acc.getBalance(),
                        transIds));
                }
            } catch (IOException e) {
                System.err.println("Error saving accounts: " + e.getMessage());
            }
        }
       
        private void saveBudgets(List<Budget> budgets) {
            try (PrintWriter writer = new PrintWriter(new FileWriter(BUDGETS_FILE))) {
                for (Budget b : budgets) {
                    writer.println(String.format("%s,%.2f,%.2f",
                        b.getCategory(),
                        b.getLimit(),
                        b.getCurrentSpending()));
                }
            } catch (IOException e) {
                System.err.println("Error saving budgets: " + e.getMessage());
            }
        }
    }

    // Service layer
    public static class FinanceService {
        private List<Account> accounts;
        private List<Budget> budgets;
        private FileDataHandler dataHandler;
        private int nextTransactionId;
       
        public FinanceService() {
            this.dataHandler = new FileDataHandler();
            List<Transaction> transactions = dataHandler.loadTransactions();
            this.accounts = dataHandler.loadAccounts(transactions);
            this.budgets = dataHandler.loadBudgets();
           
            // Calculate next transaction ID
            this.nextTransactionId = transactions.stream()
                .mapToInt(Transaction::getId)
                .max()
                .orElse(0) + 1;
        }
       
        public void saveData() {
            dataHandler.saveAllData(accounts, budgets);
        }
       
        public void addTransaction(int accountId, String description, double amount,
                                 String category, Date date, String type) {
            Transaction transaction = new Transaction(
                nextTransactionId++,
                description,
                amount,
                category,
                date,
                type);
           
            getAccountById(accountId).ifPresent(acc -> {
                acc.addTransaction(transaction);
               
                // Update budget if it's an expense
                if (type.equalsIgnoreCase("Expense")) {
                    budgets.stream()
                        .filter(b -> b.getCategory().equalsIgnoreCase(category))
                        .findFirst()
                        .ifPresent(b -> b.addSpending(amount));
                }
            });
        }
       
        public Optional<Account> getAccountById(int id) {
            return accounts.stream()
                .filter(acc -> acc.getId() == id)
                .findFirst();
        }
       
        public List<Account> getAccounts() {
            return accounts;
        }
       
        public double getTotalBalance() {
            return accounts.stream()
                .mapToDouble(Account::getBalance)
                .sum();
        }
       
        public Map<String, Double> getCategorySpending() {
            Map<String, Double> spendingByCategory = new HashMap<>();
           
            accounts.stream()
                .flatMap(acc -> acc.getTransactions().stream())
                .filter(t -> t.getType().equalsIgnoreCase("Expense"))
                .forEach(t -> {
                    spendingByCategory.merge(t.getCategory(), t.getAmount(), Double::sum);
                });
           
            return spendingByCategory;
        }
       
        public void addBudget(String category, double limit) {
            budgets.add(new Budget(category, limit));
        }
       
        public List<Budget> getBudgets() {
            return budgets;
        }
       
        public List<Transaction> getAllTransactions() {
            return accounts.stream()
                .flatMap(acc -> acc.getTransactions().stream())
                .sorted(Comparator.comparing(Transaction::getDate).reversed())
                .collect(Collectors.toList());
        }
    }

    // User Interface
    public static class FinanceConsoleUI {
        private FinanceService financeService;
        private Scanner scanner;
       
        public FinanceConsoleUI(FinanceService financeService) {
            this.financeService = financeService;
            this.scanner = new Scanner(System.in);
        }
       
        public void start() {
            System.out.println("Personal Finance Tracker");
            System.out.println("========================");
           
            while (true) {
                printMainMenu();
                int choice = getIntInput("Enter your choice: ");
               
                switch (choice) {
                    case 1:
                        addTransaction();
                        break;
                    case 2:
                        viewTransactions();
                        break;
                    case 3:
                        viewAccounts();
                        break;
                    case 4:
                        viewBudgets();
                        break;
                    case 5:
                        addBudget();
                        break;
                    case 6:
                        viewReports();
                        break;
                    case 0:
                        financeService.saveData();
                        System.out.println("Data saved. Exiting...");
                        return;
                    default:
                        System.out.println("Invalid choice. Please try again.");
                }
            }
        }
       
        private void printMainMenu() {
            System.out.println("\nMain Menu");
            System.out.println("1. Add Transaction");
            System.out.println("2. View Transactions");
            System.out.println("3. View Accounts");
            System.out.println("4. View Budgets");
            System.out.println("5. Add Budget");
            System.out.println("6. View Reports");
            System.out.println("0. Exit");
            System.out.printf("Current Total Balance: $%.2f\n", financeService.getTotalBalance());
        }
       
        private void addTransaction() {
            System.out.println("\nAdd New Transaction");
           
            // Select account
            List<Account> accounts = financeService.getAccounts();
            for (int i = 0; i < accounts.size(); i++) {
                System.out.printf("%d. %s (Balance: $%.2f)\n",
                    i+1, accounts.get(i).getName(), accounts.get(i).getBalance());
            }
            int accountChoice = getIntInput("Select account (1-" + accounts.size() + "): ", 1, accounts.size());
           
            // Transaction type
            System.out.println("1. Income");
            System.out.println("2. Expense");
            int typeChoice = getIntInput("Select type (1-2): ", 1, 2);
            String type = typeChoice == 1 ? "Income" : "Expense";
           
            // Transaction details
            String description = getStringInput("Description: ");
            double amount = getDoubleInput("Amount: ");
            String category = getStringInput("Category: ");
            Date date = getDateInput("Date (YYYY-MM-DD, today if blank): ");
           
            financeService.addTransaction(
                accounts.get(accountChoice-1).getId(),
                description,
                amount,
                category,
                date,
                type);
           
            System.out.println("Transaction added successfully!");
        }
       
        private void viewTransactions() {
            System.out.println("\nAll Transactions");
            System.out.println("ID  | Date       | Description         | Amount   | Category    | Type");
            System.out.println("----+------------+---------------------+----------+-------------+--------");
           
            List<Transaction> transactions = financeService.getAllTransactions();
            if (transactions.isEmpty()) {
                System.out.println("No transactions found.");
            } else {
                transactions.forEach(System.out::println);
            }
        }
       
        private void viewAccounts() {
            System.out.println("\nAccounts");
            financeService.getAccounts().forEach(acc -> {
                System.out.printf("%s (ID: %d)\n", acc.getName(), acc.getId());
                System.out.printf("  Balance: $%.2f\n", acc.getBalance());
                System.out.printf("  Transactions: %d\n", acc.getTransactions().size());
            });
        }
       
        private void viewBudgets() {
            System.out.println("\nBudgets");
            System.out.println("Category        | Limit     | Spent    | Remaining");
            System.out.println("----------------+-----------+----------+-----------");
           
            financeService.getBudgets().forEach(b -> {
                System.out.printf("%-15s | $%8.2f | $%7.2f | $%8.2f\n",
                    b.getCategory(),
                    b.getLimit(),
                    b.getCurrentSpending(),
                    b.getRemaining());
            });
        }
       
        private void addBudget() {
            System.out.println("\nAdd New Budget");
            String category = getStringInput("Category: ");
            double limit = getDoubleInput("Monthly limit: ");
            financeService.addBudget(category, limit);
            System.out.println("Budget added successfully!");
        }
       
        private void viewReports() {
            System.out.println("\nFinancial Reports");
            System.out.println("1. Category Spending");
            System.out.println("2. Income vs Expense");
            int choice = getIntInput("Select report (1-2): ", 1, 2);
           
            if (choice == 1) {
                System.out.println("\nCategory Spending");
                System.out.println("Category        | Amount");
                System.out.println("----------------+----------");
               
                financeService.getCategorySpending().forEach((category, amount) -> {
                    System.out.printf("%-15s | $%8.2f\n", category, amount);
                });
            } else {
                double totalIncome = financeService.getAllTransactions().stream()
                    .filter(t -> t.getType().equalsIgnoreCase("Income"))
                    .mapToDouble(Transaction::getAmount)
                    .sum();
               
                double totalExpense = financeService.getAllTransactions().stream()
                    .filter(t -> t.getType().equalsIgnoreCase("Expense"))
                    .mapToDouble(Transaction::getAmount)
                    .sum();
               
                System.out.println("\nIncome vs Expense");
                System.out.printf("Total Income:  $%.2f\n", totalIncome);
                System.out.printf("Total Expense: $%.2f\n", totalExpense);
                System.out.printf("Net Balance:   $%.2f\n", (totalIncome - totalExpense));
            }
        }
       
        // Helper methods for input
        private String getStringInput(String prompt) {
            System.out.print(prompt);
            return scanner.nextLine().trim();
        }
       
        private int getIntInput(String prompt) {
            while (true) {
                try {
                    System.out.print(prompt);
                    return Integer.parseInt(scanner.nextLine());
                } catch (NumberFormatException e) {
                    System.out.println("Please enter a valid number.");
                }
            }
        }
       
        private int getIntInput(String prompt, int min, int max) {
            while (true) {
                int value = getIntInput(prompt);
                if (value >= min && value <= max) {
                    return value;
                }
                System.out.printf("Please enter a number between %d and %d.\n", min, max);
            }
        }
       
        private double getDoubleInput(String prompt) {
            while (true) {
                try {
                    System.out.print(prompt);
                    return Double.parseDouble(scanner.nextLine());
                } catch (NumberFormatException e) {
                    System.out.println("Please enter a valid amount.");
                }
            }
        }
       
        private Date getDateInput(String prompt) {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            sdf.setLenient(false);
           
            while (true) {
                String input = getStringInput(prompt);
                if (input.isEmpty()) {
                    return new Date(); // Default to today
                }
               
                try {
                    return sdf.parse(input);
                } catch (ParseException e) {
                    System.out.println("Please enter date in YYYY-MM-DD format or leave blank for today.");
                }
            }
        }
    }

    // Main method
    public static void main(String[] args) {
        FinanceService financeService = new FinanceService();
        FinanceConsoleUI consoleUI = new FinanceConsoleUI(financeService);
       
        // Add shutdown hook to save data on exit
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            financeService.saveData();
            System.out.println("\nData saved successfully.");
        }));
       
        consoleUI.start();
    }
}
