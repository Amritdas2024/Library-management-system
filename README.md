# Library-management-system
#include <iostream>
#include <ctime>
#include <string>

using namespace std;

// Structure for User
struct User {
    int userId;
    string name;
    string email;
    User* next; // Pointer to the next user in the linked list
};

// Structure for Book
struct Book {
    int bookId;
    string title;
    string author;
    bool isAvailable;
    Book* next; // Pointer to the next book in the linked list
};

// Structure for BorrowedBook
struct BorrowedBook {
    int userId;
    int bookId;
    time_t borrowDate;
    time_t returnDate;
    bool isReturned;
    BorrowedBook* next; // Pointer to the next borrowed book in the linked list
};

// Function to add a new user
void addUser(User*& head, int userId, string name, string email) {
    User* newUser = new User{userId, name, email, nullptr};
    if (!head) {
        head = newUser;
    } else {
        User* temp = head;
        while (temp->next) {
            temp = temp->next;
        }
        temp->next = newUser;
    }
}

// Function to add a new book
void addBook(Book*& head, int bookId, string title, string author) {
    Book* newBook = new Book{bookId, title, author, true, nullptr};
    if (!head) {
        head = newBook;
    } else {
        Book* temp = head;
        while (temp->next) {
            temp = temp->next;
        }
        temp->next = newBook;
    }
}

// Function to delete a book
void deleteBook(Book*& head, int bookId) {
    if (!head) return;
    if (head->bookId == bookId) {
        Book* temp = head;
        head = head->next;
        delete temp;
        return;
    }
    Book* current = head;
    while (current->next && current->next->bookId != bookId) {
        current = current->next;
    }
    if (current->next) {
        Book* temp = current->next;
        current->next = current->next->next;
        delete temp;
    }
}

// Function to update a book's information
void updateBook(Book* head, int bookId, string newTitle, string newAuthor) {
    Book* temp = head;
    while (temp && temp->bookId != bookId) {
        temp = temp->next;
    }
    if (temp) {
        temp->title = newTitle;
        temp->author = newAuthor;
    }
}

// Function to search for a book
Book* searchBook(Book* head, int bookId) {
    Book* temp = head;
    while (temp && temp->bookId != bookId) {
        temp = temp->next;
    }
    return temp;
}

// Function to place a book requisition
void placeBookRequisition(BorrowedBook*& head, int userId, int bookId) {
    time_t now = time(0); // Get the current time
    BorrowedBook* newBorrowedBook = new BorrowedBook{userId, bookId, now, 0, false, nullptr};
    if (!head) {
        head = newBorrowedBook;
    } else {
        BorrowedBook* temp = head;
        while (temp->next) {
            temp = temp->next;
        }
        temp->next = newBorrowedBook;
    }
}

// Function to calculate fine
double calculateFine(time_t borrowDate, time_t returnDate) {
    double finePerDay = 1.0; // Assume 1 unit of currency per day
    int daysLate = difftime(returnDate, borrowDate) / (60 * 60 * 24);
    if (daysLate > 14) { // Assume 2 weeks borrowing period
        return (daysLate - 14) * finePerDay;
    }
    return 0.0;
}

// Function to return a book and calculate fine
double returnBook(BorrowedBook* head, int userId, int bookId) {
    BorrowedBook* temp = head;
    while (temp) {
        if (temp->userId == userId && temp->bookId == bookId && !temp->isReturned) {
            time_t now = time(0); // Get the current time
            temp->returnDate = now;
            temp->isReturned = true;
            return calculateFine(temp->borrowDate, temp->returnDate);
        }
        temp = temp->next;
    }
    return 0.0; // No fine if the book was not borrowed or already returned
}

// Main function
int main() {
    User* users = nullptr;
    Book* books = nullptr;
    BorrowedBook* borrowedBooks = nullptr;

    // Add some users
    addUser(users, 1, "Alice", "alice@example.com");
    addUser(users, 2, "Bob", "bob@example.com");

    // Add some books
    addBook(books, 101, "Book A", "Author A");
    addBook(books, 102, "Book B", "Author B");

    // User 1 borrows Book 101
    placeBookRequisition(borrowedBooks, 1, 101);

    // Simulate a delay before returning the book (e.g., 16 days late)
    time_t now = time(0);
    now += 16 * 24 * 60 * 60; // Add 16 days in seconds
    borrowedBooks->returnDate = now;

    // Return Book 101 and calculate fine
    double fine = returnBook(borrowedBooks, 1, 101);
    cout << "Fine for returning the book: " << fine << endl;

    return 0;
}
