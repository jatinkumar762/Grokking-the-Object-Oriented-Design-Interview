
**1. Enums**

```java
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.UUID; // For generating unique IDs

enum BookStatus {
    AVAILABLE,
    LOANED,
    RESERVED, // Added for potential extension
    LOST
}

enum AccountStatus { // Useful for Member/Librarian
    ACTIVE,
    BLOCKED,
    CLOSED
}
```

**2. Basic Data Classes**

```java
// Represents address details
class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    // Constructor, Getters
    public Address(String street, String city, String state, String zipCode) {
        this.street = street;
        this.city = city;
        this.state = state;
        this.zipCode = zipCode;
    }
    // ... Getters ...
    @Override
    public String toString() {
        return street + ", " + city + ", " + state + " " + zipCode;
    }
}

// Base class for Person details (can be inherited by Member, Librarian)
abstract class Person {
    private String name;
    private Address address;
    private String email;
    private String phone;
    // Constructor, Getters
    public Person(String name, Address address, String email, String phone) {
        this.name = name;
        this.address = address;
        this.email = email;
        this.phone = phone;
    }
     // ... Getters ...
    public String getName() { return name; }
}

// Book metadata class
class Book {
    private String isbn;
    private String title;
    private String author;
    private String subject;
    // Constructor, Getters
    public Book(String isbn, String title, String author, String subject) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
        this.subject = subject;
    }
    // ... Getters ...
    public String getIsbn() { return isbn; }
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
}
```

**3. Core Entity Classes**

```java
// Represents a specific physical copy of a book
class BookItem {
    private String barcode; // Unique ID for this physical copy
    private Book bookDetails; // Reference to the general Book info
    private BookStatus status;
    private LocalDate borrowedDate;
    private LocalDate dueDate;
    private String locationIdentifier; // e.g., Shelf number

    public BookItem(Book bookDetails, String locationIdentifier) {
        this.barcode = UUID.randomUUID().toString(); // Generate a unique barcode
        this.bookDetails = bookDetails;
        this.locationIdentifier = locationIdentifier;
        this.status = BookStatus.AVAILABLE;
    }

    // --- Getters ---
    public String getBarcode() { return barcode; }
    public Book getBookDetails() { return bookDetails; }
    public BookStatus getStatus() { return status; }
    public LocalDate getDueDate() { return dueDate; }

    // --- Setters/Updaters ---
    public void updateStatus(BookStatus newStatus) {
        this.status = newStatus;
    }

    public void setLoanDetails(LocalDate borrowedDate, LocalDate dueDate) {
        this.borrowedDate = borrowedDate;
        this.dueDate = dueDate;
        this.status = BookStatus.LOANED;
    }

     public void markAsAvailable() {
        this.status = BookStatus.AVAILABLE;
        this.borrowedDate = null;
        this.dueDate = null;
    }

    @Override
    public String toString() {
     return "Barcode: " + barcode + ", Title: " + bookDetails.getTitle() + ", Status: " + status;
    }
}

// Represents a library member
class Member extends Person {
    private String memberId;
    private AccountStatus accountStatus;
    private LocalDate dateOfMembership;
    private List<Loan> activeLoans; // Track current loans
    private double totalFineDue;

    public Member(String name, Address address, String email, String phone) {
        super(name, address, email, phone);
        this.memberId = "MEM-" + UUID.randomUUID().toString().substring(0, 8); // Simple ID generation
        this.accountStatus = AccountStatus.ACTIVE;
        this.dateOfMembership = LocalDate.now();
        this.activeLoans = new ArrayList<>();
        this.totalFineDue = 0.0;
    }

    // --- Getters ---
    public String getMemberId() { return memberId; }
    public AccountStatus getAccountStatus() { return accountStatus; }
    public int getBorrowedCount() { return activeLoans.size(); }
    public double getTotalFineDue() { return totalFineDue; }
    public List<Loan> getActiveLoans() { return activeLoans; }


    // --- Methods ---
    public void addLoan(Loan loan) {
        this.activeLoans.add(loan);
    }

    public void removeLoan(Loan loan) {
        this.activeLoans.remove(loan);
    }

     public void updateFine(double amount) {
        this.totalFineDue += amount;
    }

    public void payFine(double amount) {
         if (amount > 0 && amount <= this.totalFineDue) {
            this.totalFineDue -= amount;
            System.out.println("Paid: " + amount + ", Remaining fine: " + this.totalFineDue);
         } else {
            System.out.println("Invalid payment amount.");
         }
    }

    public void setStatus(AccountStatus status) {
        this.accountStatus = status;
    }
}

// Represents a librarian (can be extended with specific permissions)
class Librarian extends Person {
     private String librarianId;
     private AccountStatus accountStatus;

     public Librarian(String name, Address address, String email, String phone) {
        super(name, address, email, phone);
        this.librarianId = "LIB-" + UUID.randomUUID().toString().substring(0, 8);
        this.accountStatus = AccountStatus.ACTIVE;
    }
    // Librarian-specific methods would go in the Library class or a dedicated service
     public String getLibrarianId() { return librarianId; }
}
```

**4. Relationship/Tracking Classes**

```java
// Represents a loan record
class Loan {
    private String loanId;
    private BookItem bookItem;
    private Member member;
    private LocalDate issueDate;
    private LocalDate dueDate;
    private LocalDate returnDate; // Null until returned

    public Loan(BookItem bookItem, Member member, LocalDate issueDate, LocalDate dueDate) {
        this.loanId = "LOAN-" + UUID.randomUUID().toString().substring(0, 10);
        this.bookItem = bookItem;
        this.member = member;
        this.issueDate = issueDate;
        this.dueDate = dueDate;
        this.returnDate = null;
    }

    // --- Getters ---
    public String getLoanId() { return loanId; }
    public BookItem getBookItem() { return bookItem; }
    public Member getMember() { return member; }
    public LocalDate getIssueDate() { return issueDate; }
    public LocalDate getDueDate() { return dueDate; }
    public LocalDate getReturnDate() { return returnDate; }

    public void markAsReturned() {
        this.returnDate = LocalDate.now();
    }
}

// Represents a fine record (could be simplified or expanded)
class Fine {
     private String fineId;
     private Member member;
     private Loan loan; // Link fine to the specific overdue loan
     private double amount;
     private LocalDate dateIssued;
     private boolean paid;

     public Fine(Member member, Loan loan, double amount) {
        this.fineId = "FINE-" + UUID.randomUUID().toString().substring(0, 8);
        this.member = member;
        this.loan = loan;
        this.amount = amount;
        this.dateIssued = LocalDate.now();
        this.paid = false;
     }

    // --- Getters ---
     public double getAmount() { return amount; }
     public Member getMember() { return member; }

     public void markAsPaid() {
        this.paid = true;
     }
     public boolean isPaid() { return paid; }
}
```

**5. Central Orchestrator Class**

```java
// The main library facade
class Library {
    private String name;
    private Address address;
    // Using Maps for efficient lookup by ID
    private Map<String, BookItem> bookItemsCatalog; // Barcode -> BookItem
    private Map<String, Member> members;          // memberId -> Member
    private Map<String, Loan> activeLoans;        // loanId -> Loan (or could be barcode -> Loan)
    private Map<String, Fine> outstandingFines; // fineId -> Fine

    private static final int MAX_BOOKS_PER_MEMBER = 5;
    private static final int LOAN_DURATION_DAYS = 14;
    private static final double FINE_PER_DAY = 0.50;

    public Library(String name, Address address) {
        this.name = name;
        this.address = address;
        this.bookItemsCatalog = new HashMap<>();
        this.members = new HashMap<>();
        this.activeLoans = new HashMap<>();
        this.outstandingFines = new HashMap<>();
    }

    // --- Member Management ---
    public void addMember(Member member) {
        if (!members.containsKey(member.getMemberId())) {
            members.put(member.getMemberId(), member);
            System.out.println("Member added: " + member.getName());
        } else {
            System.out.println("Member ID already exists.");
        }
    }

     public void blockMember(String memberId) {
         Member member = members.get(memberId);
         if (member != null) {
             member.setStatus(AccountStatus.BLOCKED);
             System.out.println("Member blocked: " + member.getName());
         } else {
            System.out.println("Member not found.");
         }
     }
     // removeMember etc.

    // --- Book Management ---
     public void addBookItem(BookItem bookItem) {
         if (!bookItemsCatalog.containsKey(bookItem.getBarcode())) {
            bookItemsCatalog.put(bookItem.getBarcode(), bookItem);
            System.out.println("Book item added: " + bookItem.getBookDetails().getTitle() + " (" + bookItem.getBarcode() + ")");
         } else {
            System.out.println("Book item with this barcode already exists.");
         }
     }
     // removeBookItem etc.

    // --- Core Operations ---

    public List<BookItem> searchBook(String query) {
        // Simple search by title or author (can be made more sophisticated)
        List<BookItem> results = new ArrayList<>();
        String lowerQuery = query.toLowerCase();
        for (BookItem item : bookItemsCatalog.values()) {
            if (item.getBookDetails().getTitle().toLowerCase().contains(lowerQuery) ||
                item.getBookDetails().getAuthor().toLowerCase().contains(lowerQuery)) {
                results.add(item);
            }
        }
         System.out.println("Search results for '" + query + "': " + results.size() + " items found.");
        return results;
    }

    public boolean issueBook(String memberId, String barcode) {
        Member member = members.get(memberId);
        BookItem bookItem = bookItemsCatalog.get(barcode);

        // 1. Validations
        if (member == null) {
            System.out.println("Error: Member not found.");
            return false;
        }
        if (bookItem == null) {
            System.out.println("Error: Book item not found.");
            return false;
        }
         if (member.getAccountStatus() != AccountStatus.ACTIVE) {
            System.out.println("Error: Member account is not active ("+ member.getAccountStatus() +").");
            return false;
         }
         if (member.getBorrowedCount() >= MAX_BOOKS_PER_MEMBER) {
            System.out.println("Error: Member has reached the borrowing limit (" + MAX_BOOKS_PER_MEMBER + ").");
            return false;
         }
         if (bookItem.getStatus() != BookStatus.AVAILABLE) {
            System.out.println("Error: Book is not available ("+ bookItem.getStatus() +").");
            return false;
         }

         // 2. Create Loan and Update Status
         LocalDate issueDate = LocalDate.now();
         LocalDate dueDate = issueDate.plusDays(LOAN_DURATION_DAYS);
         Loan newLoan = new Loan(bookItem, member, issueDate, dueDate);

         bookItem.setLoanDetails(issueDate, dueDate); // Updates status to LOANED
         member.addLoan(newLoan);
         activeLoans.put(newLoan.getLoanId(), newLoan); // Track the active loan

         System.out.println("Book '" + bookItem.getBookDetails().getTitle() + "' issued to " + member.getName() + ". Due date: " + dueDate);
        return true;
    }


    public boolean returnBook(String barcode) {
         BookItem bookItem = bookItemsCatalog.get(barcode);
         if (bookItem == null || bookItem.getStatus() != BookStatus.LOANED) {
             System.out.println("Error: Book item not found or not on loan.");
             return false;
         }

        // Find the active loan for this book item
        Loan loanToReturn = null;
        for (Loan loan : activeLoans.values()) {
             if (loan.getBookItem().getBarcode().equals(barcode) && loan.getReturnDate() == null) {
                loanToReturn = loan;
                break;
             }
        }

         if (loanToReturn == null) {
            System.out.println("Error: Active loan record not found for barcode " + barcode);
            return false; // Should not happen if status is LOANED, but good check
         }

         Member member = loanToReturn.getMember();
         loanToReturn.markAsReturned(); // Set return date

         // Calculate Fine
         LocalDate today = LocalDate.now();
         if (today.isAfter(loanToReturn.getDueDate())) {
             long daysOverdue = java.time.temporal.ChronoUnit.DAYS.between(loanToReturn.getDueDate(), today);
             double fineAmount = daysOverdue * FINE_PER_DAY;
             Fine fine = new Fine(member, loanToReturn, fineAmount);
             outstandingFines.put(fine.getFineId(), fine);
             member.updateFine(fineAmount); // Add to member's total due
             System.out.println("Book returned late. Fine added: $" + fineAmount + ". Total due: $" + member.getTotalFineDue());
         } else {
             System.out.println("Book '" + bookItem.getBookDetails().getTitle() + "' returned on time by " + member.getName() + ".");
         }

         // Update statuses
         bookItem.markAsAvailable(); // Set status to AVAILABLE, clear dates
         member.removeLoan(loanToReturn); // Remove from member's active list
         activeLoans.remove(loanToReturn.getLoanId()); // Remove from library's active loans

         return true;
    }

     public void collectFine(String memberId, double amount) {
         Member member = members.get(memberId);
         if (member != null) {
             if (member.getTotalFineDue() > 0) {
                // In a real system, you'd likely apply payment to specific fine records
                member.payFine(amount);
                // Potentially update Fine objects in outstandingFines map to paid status
             } else {
                System.out.println(member.getName() + " has no outstanding fines.");
             }
         } else {
            System.out.println("Member not found.");
         }
     }

} // End of Library class
```

**How to Use (Conceptual Example):**

```java
public class LibrarySystemDemo {
    public static void main(String[] args) {
        // 1. Setup Library
        Address libraryAddress = new Address("123 Library St", "Bookville", "StateRead", "12345");
        Library mainLibrary = new Library("City Central Library", libraryAddress);

        // 2. Add Books
        Book book1 = new Book("978-1234567890", "The Great Adventure", "Jane Doe", "Fiction");
        Book book2 = new Book("978-0987654321", "Learning Java", "John Smith", "Programming");
        BookItem item1 = new BookItem(book1, "Shelf A1");
        BookItem item2 = new BookItem(book1, "Shelf A2"); // Another copy
        BookItem item3 = new BookItem(book2, "Shelf B5");
        mainLibrary.addBookItem(item1);
        mainLibrary.addBookItem(item2);
        mainLibrary.addBookItem(item3);

        // 3. Add Members
        Address memberAddress = new Address("45 Resident Ave", "Bookville", "StateRead", "12346");
        Member member1 = new Member("Alice Wonderland", memberAddress, "alice@example.com", "555-1111");
        mainLibrary.addMember(member1);

        // 4. Search
        mainLibrary.searchBook("Adventure");
        mainLibrary.searchBook("Java");

        // 5. Issue a Book
        mainLibrary.issueBook(member1.getMemberId(), item1.getBarcode());
        System.out.println(member1.getName() + " has borrowed " + member1.getBorrowedCount() + " books.");

        // 6. Try to issue unavailable book
        mainLibrary.issueBook(member1.getMemberId(), item1.getBarcode()); // Should fail

        // 7. Return a Book (assuming time passed and it's now late)
        // To simulate lateness, you'd manipulate dates or inject a Clock dependency
        // For simplicity, let's assume returnBook calculates based on current date vs due date
         System.out.println("\n--- Simulating Return ---");
        // Pretend due date was yesterday for testing fine calculation
        // In real code, avoid direct manipulation like this
         if(item1.getStatus() == BookStatus.LOANED) { // Check needed as issue might fail
             LocalDate yesterday = LocalDate.now().minusDays(1);
             // Temporary hack for demo - DO NOT do this in real code
             try {
                 java.lang.reflect.Field dueDateField = BookItem.class.getDeclaredField("dueDate");
                 dueDateField.setAccessible(true);
                 dueDateField.set(item1, yesterday);
             } catch (Exception e) { e.printStackTrace(); }
         }
        mainLibrary.returnBook(item1.getBarcode());


        // 8. Collect Fine
        mainLibrary.collectFine(member1.getMemberId(), 0.50);
        mainLibrary.collectFine(member1.getMemberId(), 1.00); // Try paying more than due

         System.out.println("\nFinal Status:");
         System.out.println(item1); // Show status is AVAILABLE again
         System.out.println(member1.getName() + " fine due: $" + member1.getTotalFineDue());

    }
}
```

Remember, this code focuses on the object structure and basic interactions. A production system would need proper error handling, potentially database integration, more robust ID generation, concurrency control, better configuration management (like fine rates), and a user interface layer.