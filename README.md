# Library-Management-System

import java.io.*;
import java.util.*;

class Book implements Serializable 
{
    String title;
    String author;
    String isbn;
    boolean isBorrowed;

    public Book(String title, String author, String isbn) {
        this.title = title;
        this.author = author;
        this.isbn = isbn;
        this.isBorrowed = false; // Default: not borrowed
    }

    

    // Method to display book details (for users)
    public void displayDetails() {
        System.out.println("Title: " + title);
        System.out.println("Author: " + author);
        System.out.println("ISBN: " + isbn);
        System.out.println("Status: " + (isBorrowed ? "Borrowed" : "Available"));
    }
}

class Library implements Serializable 
{
    private List<Book> bookList; // Dynamic storage for books
    private Stack<String> actionStack; // Stack for undo functionality
    private LinkedList<Book> recentBooks; // LinkedList for recently added books
    private Map<String, String> borrowedBooks; // To keep track of who borrowed which book

    public Library() 
    {
        bookList = new ArrayList<>();
        actionStack = new Stack<>();
        recentBooks = new LinkedList<>();
        borrowedBooks = new HashMap<>();
    }
    // Save the library state to a file
    public void saveToFile(String filename) 
    {
        try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(filename))) 
        {
            oos.writeObject(this);
            System.out.println("Library state saved to " + filename);
        } catch (IOException e) 
        {
            System.out.println("Error saving library state: " + e.getMessage());
        }
    }

    // Load the library state from a file
    public static Library loadFromFile(String filename) 
    {
        try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream(filename))) 
        {
            return (Library) ois.readObject();
        } catch (IOException | ClassNotFoundException e) 
        {
            System.out.println("Error loading library state: " + e.getMessage());
            return new Library();
        }
    }



    // Method to add a book
    public void addBook(String title, String author, String isbn) 
    {
        Book book = new Book(title, author, isbn);
        bookList.add(book);
        actionStack.push("Removed: " + title); // Store action for undo
        recentBooks.add(book); // Add to recent books
        System.out.println("Book added: " + title);
    }

    // Method to remove a book
    public void removeBook(String title) 
    {
        for (Book book : bookList) 
        {
            if (book.title.equals(title)) 
            {
                bookList.remove(book);
                actionStack.push("Added: " + title); // Store action for undo
                System.out.println("Book removed: " + title);
                return;
            }
        }
        System.out.println("Book not found: " + title);
    }

    // Method to display all books
    public void displayBooks() 
    {
        System.out.println("Books in the Library:");
        if (bookList.isEmpty()) 
        {
            System.out.println("No books available.");
        } 
        else 
        {
            int i = 1;
            for (Book book : bookList) 
            {
                System.out.println(i + ". " + book.title + " by " + book.author + " (ISBN: " + book.isbn + ") - " 
                    + (book.isBorrowed ? "Borrowed" : "Available"));
                i++;
            }
        }
    }

    // Method to undo the last action
    public void undo() 
    {
        if (!actionStack.isEmpty()) 
        {
            String lastAction = actionStack.pop();
            if (lastAction.startsWith("Removed: ")) 
            {
                String title = lastAction.substring(9);
                System.out.println("Undo: Added " + title);
                // Use loop to remove the book
                Iterator<Book> iterator = bookList.iterator();
                while (iterator.hasNext()) 
                {
                    Book book = iterator.next();
                    if (book.title.equals(title)) 
                    {
                        iterator.remove();
                        break;
                    }
                }
            } 
            else if (lastAction.startsWith("Added: ")) 
            {
                String title = lastAction.substring(7);
                System.out.println("Undo: Removed " + title);
                bookList.add(recentBooks.removeLast());
            }
        } 
        else 
        {
            System.out.println("No actions to undo.");
        }
    }

    // Method for users to select a book and view details
    public void selectBook(int bookIndex) 
    {
        if (bookIndex >= 0 && bookIndex < bookList.size()) 
        {
            Book selectedBook = bookList.get(bookIndex);
            selectedBook.displayDetails();
        } 
        else 
        {
            System.out.println("Invalid book selection.");
        }
    }

    // Method for users to take a book (borrow it)
    public void takeBook(int bookIndex, String username) 
    {
        if (bookIndex >= 0 && bookIndex < bookList.size()) 
        {
            Book book = bookList.get(bookIndex);
            if (!book.isBorrowed) 
            {
                book.isBorrowed = true; // Mark as borrowed
                borrowedBooks.put(book.title, username); // Store who borrowed the book
                System.out.println(username + " has borrowed the book: " + book.title);
            } 
            else 
            {
                System.out.println("Sorry, this book is already borrowed.");
            }
        } 
        else 
        {
            System.out.println("Invalid book selection.");
        }
    }

    // Method for users to return a book
    public void returnBook(int bookIndex, String username) 
    {
        if (bookIndex >= 0 && bookIndex < bookList.size()) 
        {
            Book book = bookList.get(bookIndex);
            if (book.isBorrowed && borrowedBooks.get(book.title).equals(username)) 
            {
                book.isBorrowed = false; // Mark as available
                borrowedBooks.remove(book.title); // Remove from borrowed list
                System.out.println(username + " has returned the book: " + book.title);
            } 
            else 
            {
                System.out.println("This book was not borrowed by you.");
            }
        } 
        else 
        {
            System.out.println("Invalid book selection.");
        }
    }
}

class Admin 
{
    private String username;
    private String password;

    public Admin(String username, String password) 
    {
        this.username = username;
        this.password = password;
    }

    // Method to authenticate the admin
    public boolean login(String enteredUsername, String enteredPassword) 
    {
        return username.equals(enteredUsername) && password.equals(enteredPassword);
    }
}

public class Main
{
    private static final String LIBRARY_FILE = "library.dat";
    private static Library library = Library.loadFromFile(LIBRARY_FILE);
    private static Admin admin = new Admin("admin", "123"); // Admin credentials
    private static Scanner scanner = new Scanner(System.in);
    private static boolean isAuthenticated = false;
    private static boolean isAdmin = false;
    private static String username = "";

    public static void main(String[] args)
    {
        // Save the library state on exit
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            library.saveToFile(LIBRARY_FILE);
        }));

        // Role selection prompt
        boolean check=false;
        while(check==false)
        {
            try
            {
                System.out.println("Welcome to the Library Management System");
                System.out.println("Are you logging in as:");
                System.out.println("1. Admin");
                System.out.println("2. User");
                System.out.print("Enter your choice: ");
                int roleChoice = scanner.nextInt();
                scanner.nextLine(); // Consume newline

                if (roleChoice == 1) 
                {
                    // Admin login
                    boolean check1=false;
                    while(check1==false)
                    {
                        System.out.print("Enter username: ");
                        String enteredUsername = scanner.nextLine();
                        System.out.print("Enter password: ");
                        String enteredPassword = scanner.nextLine();

                        // Authenticate admin
                        if (admin.login(enteredUsername, enteredPassword)) 
                        {
                            System.out.println("Login successful. Welcome, Admin!");
                            isAuthenticated = true;
                            isAdmin = true; // Mark as admin
                            check1=true;
                        } 
                        else 
                        {
                            System.out.println("Invalid credentials. Access denied.");
                        }
                    }
                    
                } 
                else if (roleChoice == 2) 
                {
                    // User role (no authentication needed)
                    boolean check2=false;
                    while(check2==false)
                    {
                        System.out.print("Enter your username: ");
                        username = scanner.nextLine(); // Ask for username
                        if(username.matches("[a-zA-z\\s]+"))
                        {
                        System.out.println("Logged in as User: " + username);
                        isAuthenticated = true; // Users are always authenticated
                        isAdmin = false; // Mark as user
                        check2=true;
                        }
                        else
                        {
                            System.out.println("Invalid username. Please enter a valid username.");
                        }
                    }
                } 
                else
                {
                    System.out.println("Invalid choice. Try again.");
                }
        
                if (isAuthenticated) 
                {
                    // Main menu for admin and user
                    do 
                    {
                        if (isAdmin) 
                        {
                            System.out.println("\nAdmin Library Management System");
                            System.out.println("1. Add Book");
                            System.out.println("2. Remove Book");
                            System.out.println("3. Display All Books");
                            System.out.println("4. Undo Last Action");
                            System.out.println("5. back");
                            System.out.println("6. Exit");
                            System.out.print("Enter your choice: ");
                            try {
                                roleChoice = scanner.nextInt();
                                scanner.nextLine(); // Consume newline

                                switch (roleChoice) 
                                {
                                    case 1:
                                        System.out.print("Enter title: ");
                                        String title = scanner.nextLine();
                                        System.out.print("Enter author: ");
                                        String author = scanner.nextLine();
                                        System.out.print("Enter ISBN: ");
                                        String isbn = scanner.nextLine();
                                        if(title.matches("[a-zA-z\\s]+") && author.matches("[a-zA-z\\s]+") && isbn.matches("[0-9]+"))
                                        {
                                           library.addBook(title, author, isbn);
                                           break;
                                        }
                                    case 2:
                                        System.out.print("Enter title to remove: ");
                                        title = scanner.nextLine();
                                        if(title.matches("[a-zA-z\\s]+"))
                                        {
                                          library.removeBook(title);
                                          break;
                                        }
                                    case 3:
                                        library.displayBooks();
                                        break;
                                    case 4:
                                        library.undo();
                                        break;
                                    case 5:
                                        System.out.println("waiting for next input");
                                        break;
                                    case 6:
                                        System.out.println("Exiting...");
                                        check=true; 
                                        roleChoice=5;
                                        break;
                                    default:
                                        System.out.println("Invalid choice. Try again.");
                                }
                            } catch (InputMismatchException e) {
                                System.out.println("Invalid input. Please enter a valid choice.");
                                scanner.nextLine(); // Consume newline
                            }
                        } 
                        else 
                        {
                            // User can view, take, and return books
                            try
                            {
                                System.out.println("\nUser Library System");
                                System.out.println("1. Display All Books");
                                System.out.println("2. Select a Book to View Details");
                                System.out.println("3. Take (Borrow) a Book");
                                System.out.println("4. Return a Book");
                                System.out.println("5. back");
                                System.out.println("6. Exit");
                                System.out.print("Enter your choice: ");
                                roleChoice = scanner.nextInt();
                                scanner.nextLine(); // Consume newline

                                switch (roleChoice) 
                                {
                                    case 1:
                                    library.displayBooks();
                                    break;
                                    case 2:
                                        library.displayBooks(); // Show available books for user to select
                                        System.out.print("Enter the book number to view details: ");
                                        int bookIndex = scanner.nextInt() - 1; // Index starts from 0
                                        library.selectBook(bookIndex); // User selects a book
                                        break;
                                    case 3:
                                        library.displayBooks(); // Show available books for borrowing
                                        System.out.print("Enter the book number to take: ");
                                        bookIndex = scanner.nextInt() - 1;
                                        library.takeBook(bookIndex, username); // User borrows the book
                                        break;
                                    case 4:
                                        library.displayBooks(); // Show books (user must know which one they borrowed)
                                        System.out.print("Enter the book number to return: ");
                                        bookIndex = scanner.nextInt() - 1;
                                        library.returnBook(bookIndex, username); // User returns the book
                                        break;
                                    case 5:
                                        System.out.println("waiting for next input");
                                        break;
                                    case 6:
                                        System.out.println("Exiting...");
                                        check=true;
                                        roleChoice=5;
                                        break;
                                    default:
                                        System.out.println("Invalid choice. Try again.");
                                }
                            }
                            catch(InputMismatchException e)
                            {
                                System.out.println("Invalid input. Please enter a valid choice.");
                                scanner.nextLine(); // Consume newline
                            } 
                        } 
                    }  
                    while (roleChoice != 5); // back condition for both admin and user
                }
            }
            catch(InputMismatchException e)
            {
                System.out.println("Invalid input. Please enter a valid choice.");
                scanner.nextLine(); // Consume newline
            }        
        }
    }    
}


