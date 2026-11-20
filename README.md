"""SpendWise: Personal Expense Tracker (single-file console program)."""
import json
import math
from datetime import datetime

DATA_FILE = "expenses.json"
DATE_FORMAT = "%Y-%m-%d"
# Tuple: the allowed categories are fixed, so they should not change
CATEGORIES = ("Food", "Transportation", "School Supplies", "Entertainment", "Others")


# ---------- Calculations ----------
def add_expense(expenses, description, amount, category, date):
    """Check every field, then add the expense (a dictionary) to the list."""
    description = description.strip()
    if not description:
        raise ValueError("Description is required")
    if not math.isfinite(amount) or amount <= 0:
        raise ValueError("Amount must be a finite number greater than zero")
    if category not in CATEGORIES:
        raise ValueError(f"Unknown category: {category}")
    try:
        datetime.strptime(date, DATE_FORMAT)
    except ValueError:
        raise ValueError("Date must be in YYYY-MM-DD format")
    expenses.append({"description": description, "amount": amount,
                     "category": category, "date": date})


def total_spent(expenses):
    """Return the sum of all expense amounts."""
    return sum(item["amount"] for item in expenses)


def totals_by_category(expenses):
    """Return a dictionary of total spending per category."""
    totals = {}
    for item in expenses:
        totals[item["category"]] = totals.get(item["category"], 0) + item["amount"]
    return totals


def remaining_balance(budget, expenses):
    """Return budget minus total spent (a negative number means overspent)."""
    return budget - total_spent(expenses)


# ---------- File handling ----------
def save_data(budget, expenses, filename=DATA_FILE):
    """Write the budget and expenses to a JSON file."""
    with open(filename, "w", encoding="utf-8") as file:
        json.dump({"budget": budget, "expenses": expenses}, file, indent=2)


def load_data(filename=DATA_FILE):
    """Return (budget, expenses). A missing file gives (0.0, [])."""
    try:
        with open(filename, "r", encoding="utf-8") as file:
            data = json.load(file)
    except FileNotFoundError:
        return 0.0, []
    except json.JSONDecodeError:
        raise ValueError("Data file is corrupted (not valid JSON)")
    if (not isinstance(data, dict) or "budget" not in data
            or not isinstance(data.get("expenses"), list)):
        raise ValueError("Data file has an unexpected structure")
    return data["budget"], data["expenses"]


# ---------- User interface ----------
def read_number(prompt):
    """Keep asking until the user types a valid number."""
    while True:
        try:
            return float(input(prompt))
        except ValueError:
            print("Invalid number. Please try again.")


def set_budget():
    """Ask for a budget greater than zero."""
    while True:
        budget = read_number("Enter your budget: ")
        if budget > 0:
            return budget
        print("Budget must be greater than zero.")


def choose_category():
    """Show the categories and return the one the user picks."""
    for number, name in enumerate(CATEGORIES, start=1):
        print(f"  {number}. {name}")
    while True:
        choice = input("Choose category number: ").strip()
        if choice.isdigit() and 1 <= int(choice) <= len(CATEGORIES):
            return CATEGORIES[int(choice) - 1]
        print("Invalid choice.")


def add_expense_menu(expenses):
    """Ask for the expense details and try to add them."""
    description = input("Description: ")
    amount = read_number("Amount: ")
    category = choose_category()
    today = datetime.now().strftime(DATE_FORMAT)
    date = input(f"Date (YYYY-MM-DD) [{today}]: ").strip() or today
    try:
        add_expense(expenses, description, amount, category, date)
        print("Expense recorded.")
    except ValueError as error:
        print("Could not add expense:", error)


def view_expenses(expenses):
    """Print all expenses as a numbered list."""
    if not expenses:
        print("No expenses recorded yet.")
        return
    for number, item in enumerate(expenses, start=1):
        print(f"{number}. {item['date']} | {item['category']} | "
              f"{item['description']} | {item['amount']:.2f}")


def show_summary(budget, expenses):
    """Print totals, remaining balance, and an over-budget warning."""
    print(f"Budget: {budget:.2f}")
    print(f"Total spent: {total_spent(expenses):.2f}")
    for category, total in totals_by_category(expenses).items():
        print(f"  {category}: {total:.2f}")
    remaining = remaining_balance(budget, expenses)
    print(f"Remaining: {remaining:.2f}")
    if remaining < 0:
        print("WARNING: You have exceeded your budget!")


def main():
    try:
        budget, expenses = load_data()
    except ValueError as error:
        print("Problem loading saved data:", error)
        print("Starting with empty records.")
        budget, expenses = 0.0, []
    if budget <= 0:
        budget = set_budget()

    while True:
        print("\n=== SpendWise ===")
        print("1. Add expense\n2. View expenses\n3. Summary\n"
              "4. Change budget\n5. Save and exit")
        choice = input("Choose: ").strip()
        if choice == "1":
            add_expense_menu(expenses)
        elif choice == "2":
            view_expenses(expenses)
        elif choice == "3":
            show_summary(budget, expenses)
        elif choice == "4":
            budget = set_budget()
        elif choice == "5":
            save_data(budget, expenses)
            print("Data saved. Goodbye!")
            break
        else:
            print("Invalid menu choice.")


if __name__ == "__main__":
    main()
