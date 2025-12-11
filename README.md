
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_TITLE 100
#define MAX_AUTHOR 100
#define TABLE_SIZE 27

struct Book {
    int id;
    char title[MAX_TITLE];
    char author[MAX_AUTHOR];
    int copies;
    struct Book *next;
};

struct Book* hashTable[TABLE_SIZE];

int hash(char *title) {
    char c = title[0];
    if (c >= 'A' && c <= 'Z') return c - 'A';
    if (c >= 'a' && c <= 'z') return c - 'a';
    return 26;
}

struct Book* createBook(int id, char *title, char *author, int copies) {
    struct Book* newBook = (struct Book*)malloc(sizeof(struct Book));
    newBook->id = id;
    strcpy(newBook->title, title);
    strcpy(newBook->author, author);
    newBook->copies = copies;
    newBook->next = NULL;
    return newBook;
}

void saveToFile() {
    FILE *fp = fopen("library.txt", "w");
    if (!fp) return;

    for (int i = 0; i < TABLE_SIZE; i++) {
        struct Book *temp = hashTable[i];
        while (temp) {
            fprintf(fp, "%d|%s|%s|%d\n", temp->id, temp->title, temp->author, temp->copies);
            temp = temp->next;
        }
    }
    fclose(fp);
}

void loadFromFile() {
    FILE *fp = fopen("library.txt", "r");
    if (!fp) return;

    char line[300];
    while (fgets(line, sizeof(line), fp)) {
        int id, copies;
        char title[MAX_TITLE], author[MAX_AUTHOR];

        char *token = strtok(line, "|");
        if (!token) continue;
        id = atoi(token);

        token = strtok(NULL, "|");
        if (!token) continue;
        strcpy(title, token);

        token = strtok(NULL, "|");
        if (!token) continue;
        strcpy(author, token);

        token = strtok(NULL, "|\n");
        if (!token) continue;
        copies = atoi(token);

        int index = hash(title);
        struct Book *newBook = createBook(id, title, author, copies);

        newBook->next = hashTable[index];
        hashTable[index] = newBook;
    }

    fclose(fp);
}

struct Book* searchBook(char *title) {
    int index = hash(title);
    struct Book *temp = hashTable[index];

    while (temp) {
        if (strcmp(temp->title, title) == 0) return temp;
        temp = temp->next;
    }
    return NULL;
}

void orderBook(char *title, int qty) {
    struct Book *book = searchBook(title);
    if (!book) {
        printf("Book NOT found. Cannot order.\n");
        return;
    }

    if (book->copies >= qty) {
        book->copies -= qty;
        printf("%d copy/copies ordered! Remaining copies: %d\n", qty, book->copies);
        saveToFile();
    } else {
        printf("Only %d copies available! Cannot order %d.\n", book->copies, qty);
    }
}

void returnBook(char *title, int qty) {
    struct Book *book = searchBook(title);
    if (!book) {
        printf("Book NOT found. Cannot return.\n");
        return;
    }
    book->copies += qty;
    printf("%d copy/copies returned! Available copies: %d\n", qty, book->copies);
    saveToFile();
}

void addBook(int id, char *title, char *author, int copies) {
    int index = hash(title);
    struct Book *newBook = createBook(id, title, author, copies);

    newBook->next = hashTable[index];
    hashTable[index] = newBook;
    saveToFile();

    printf("Book added successfully!\n");
}

void displayBooks() {
    printf("\n====== LIBRARY BOOK LIST ======\n");

    for (int i = 0; i < TABLE_SIZE; i++) {
        struct Book *temp = hashTable[i];
        if (temp) printf("[%c] -> ", (i < 26 ? 'A' + i : '#'));
        while (temp) {
            printf("%s (Copies: %d) -> ", temp->title, temp->copies);
            temp = temp->next;
        }
        if (hashTable[i]) printf("NULL\n");
    }
}

int main() {
    int choice, id, copies, qty;
    char title[MAX_TITLE], author[MAX_AUTHOR];

    for (int i = 0; i < TABLE_SIZE; i++) hashTable[i] = NULL;

    loadFromFile();

    while (1) {
        printf("\n===== BOOK ORDERING APP =====\n");
        printf("1. Add New Book\n");
        printf("2. Search Book\n");
        printf("3. Order Book\n");
        printf("4. Return Book\n");
        printf("5. Display All Books\n");
        printf("6. Exit\n");
        printf("7. Save to File\n");
        printf("Enter choice: ");
        scanf("%d", &choice);
        getchar();

        switch (choice) {
            case 1:
                printf("Enter ID: ");
                scanf("%d", &id);
                getchar();

                printf("Enter Title: ");
                fgets(title, MAX_TITLE, stdin);
                title[strcspn(title, "\n")] = 0;

                printf("Enter Author: ");
                fgets(author, MAX_AUTHOR, stdin);
                author[strcspn(author, "\n")] = 0;

                printf("Enter Copies: ");
                scanf("%d", &copies);

                addBook(id, title, author, copies);
                break;

            case 2: {
                printf("Enter Title: ");
                fgets(title, MAX_TITLE, stdin);
                title[strcspn(title, "\n")] = 0;

                struct Book* b = searchBook(title);
                if (b)
                    printf("FOUND! %s - Copies: %d\n", b->title, b->copies);
                else
                    printf("Book NOT found.\n");
                break;
            }

            case 3:
                printf("Enter Title to Order: ");
                fgets(title, MAX_TITLE, stdin);
                title[strcspn(title, "\n")] = 0;

                printf("How many copies do you need?: ");
                scanf("%d", &qty);
                getchar();

                orderBook(title, qty);
                break;

            case 4:
                printf("Enter Title to Return: ");
                fgets(title, MAX_TITLE, stdin);
                title[strcspn(title, "\n")] = 0;

                printf("How many copies are you returning?: ");
                scanf("%d", &qty);
                getchar();

                returnBook(title, qty);
                break;

            case 5:
                displayBooks();
                break;

            case 6:
                printf("Exiting...\n");
                exit(0);

            case 7:
                saveToFile();
                printf("File saved manually!\n");
                break;

            default:
                printf("Invalid choice!\n");
        }
    }
}
