# HW8 - Linked List
## Problem 1:
Write a program in C to create a singly linked list of n nodes and display the nodes in original
and reversed orders. Your program should first ask user for integers. In a loop, it reads in integers
scanf, creates structure nodes malloc, saves the integers into the nodes, and links the
nodes onto a linked list. To finish inputting integers, the user presses ctrl-d, and scanf() will
return EOF (e.g., if(scanf("%d", ...) == EOF) break; )
Your program prints the integers from the beginning of the list to the end, and then it prints the
integers in reverse order (i.e., from the end to the beginning).

>Hint: To print the integers in reverse order, you can create a new linked list: remove the nodes on
the original list from beginning to end, and add then to the front of the new list. In this way, the
last node in the original list will become the first node in the new list, the second last becomes the
second, ... Your program can display the nodes on the new list. Do not use an array to save the
data in the original list, and then display the data in the array in reversed order.

- HW solution:
```c
#include <stdio.h>
#include <stdlib.h>

struct node {
    int value;
    struct node *next;
};

int main() {
    struct node *head = NULL, *tail = NULL, *rev = NULL, *newNode, *temp;
    int x, count = 1;

    while (1) {
        printf("Input data for node %d : ", count);
        if (scanf("%d", &x) == EOF) break;

        newNode = malloc(sizeof(struct node));
        newNode->value = x;
        newNode->next = NULL;

        if (head == NULL) {
            head = newNode;
            tail = newNode;
        } else {
            tail->next = newNode;
            tail = newNode;
        }
        count++;
    }

    printf("\nData entered in the list are:\n");
    temp = head;
    while (temp != NULL) {
        printf("Data = %d\n", temp->value);
        temp = temp->next;
    }

    while (head != NULL) {
        temp = head;
        head = head->next;
        temp->next = rev;
        rev = temp;
    }

    printf("\nThe list in reverse are:\n");
    temp = rev;
    while (temp != NULL) {
        printf("Data = %d\n", temp->value);
        temp = temp->next;
    }

    return 0;
}
```

- Usage example:
```bash
$ gcc hw8.c -o hw8
$ ./hw8
Input data for node 1 : 10
Input data for node 2 : 20
Input data for node 3 : 30
Input data for node 4 : ^D
Data entered in the list are:
Data = 10
Data = 20
Data = 30
The list in reverse are:
Data = 30
Data = 20
Data = 10
```

## Problem 2:
Write a C program that use bubble sort to sort the integer values provided through user inputs (not from
the command line). Your program should first ask user for integers. In a loop, it reads in integers
(scanf), creates structure nodes (malloc), saves the integers into the nodes, and links the nodes onto a
linked list.
To finish inputting integers, the user presses ctrl-d, and scanf() will return EOF. Then, your program sorts
the nodes on the list by moving the nodes on the list. The nodes saving smaller values are moved to the
front and the nodes saving larger values are moved to the rear.
Use bubble sort. Do not use radix sort or other sort algorithms. Sort the nodes on the linked list directly.
Do not use an array to save and sort the values. I know you can swap the values in the nodes to keep them
in ascending order. But this problem is for you to practice. Thus, your program is not allowed to change
the value in a node after the value is initialized. To sort the values, your program can only change the
positions of the nodes on the list (i.e., unlink a node from the list and relink it to somewhere else).
When sorting is finished, your program prints out the integers saved in the nodes from the first node to the
last node on the list, one number on each line.
Test your program manually first. Manually type in inputs, press ctrl-d, and check output. To fully test
your program with more numbers, modify and use the following scripts. $1 of the script is the count of
the integer values. Take screenshots when you test your program manually, such that we can see the
numbers provided to the program and the output of the program.

- HW solution:
```c
#include <stdio.h>
#include <stdlib.h>

struct node {
    int value;
    struct node *next;
};

int main() {
    struct node *head = NULL, *tail = NULL, *newNode, *temp;
    int x;

    fprintf(stderr, "Enter integers (Ctrl-D to finish):\n");

    while (scanf("%d", &x) == 1) {
        newNode = malloc(sizeof(struct node));
        newNode->value = x;
        newNode->next = NULL;

        if (head == NULL) {
            head = newNode;
            tail = newNode;
        } else {
            tail->next = newNode;
            tail = newNode;
        }
    }

    if (head != NULL) {
        int swapped;
        do {
            swapped = 0;
            struct node **pp = &head;

            while (*pp && (*pp)->next) {
                struct node *a = *pp;
                struct node *b = a->next;

                if (a->value > b->value) {
                    a->next = b->next;
                    b->next = a;
                    *pp = b;
                    swapped = 1;
                }
                pp = &((*pp)->next);
            }
        } while (swapped);
    }

    temp = head;
    while (temp) {
        printf("%d\n", temp->value);
        temp = temp->next;
    }

    temp = head;
    while (temp) {
        struct node *next = temp->next;
        free(temp);
        temp = next;
    }

    return 0;
}
```
- Output example:
```bash
$ gcc hw8_sort.c -o hw8_sort
$ ./hw8_sort
Enter integers (Ctrl-D to finish):
5
3
8
1
1
^D
1
1
3
5
8
```
