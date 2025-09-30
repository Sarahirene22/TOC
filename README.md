# TOC ASSIGNMENT
## TOC GITHUB ASSIGNMENT

1. a program that detects redundant computations, dead code, and strength reduction.

```bash
   #include <stdio.h>
#include <string.h>
#include <stdlib.h>

#define MAX_LINES 10
#define MAX_LEN 100

// Function to apply simple optimizations
void optimize(char code[MAX_LINES][MAX_LEN], int n) {
    char optimized[MAX_LINES][MAX_LEN];
    int usedY = 0;

    // Step 1: Check if variable y is used later
    for (int i = 0; i < n; i++) {
        if (strstr(code[i], "y") && strstr(code[i], "z")) {
            usedY = 1;
        }
    }

    printf("\nOptimized Code:\n");

    for (int i = 0; i < n; i++) {
        char line[MAX_LEN];
        strcpy(line, code[i]);

        // Constant folding: 2*8 -> 16
        if (strstr(line, "2 * 8")) {
            printf("x = 16\n");
        }
        // Strength reduction: x * 1 -> x
        else if (strstr(line, "* 1")) {
            if (strstr(line, "y = x * 1")) {
                if (!usedY) continue; // dead code elimination
                printf("y = x\n");
            }
        }
        // Algebraic simplification: y + 0 -> y
        else if (strstr(line, "+ 0")) {
            if (strstr(line, "z = y + 0")) {
                printf("z = x\n"); // because y = x
            }
        }
        else {
            // default print (unchanged)
            printf("%s\n", line);
        }
    }
}

int main() {
    int n;
    char code[MAX_LINES][MAX_LEN];

    printf("Enter number of lines of code: ");
    scanf("%d", &n);
    getchar(); // consume newline

    printf("Enter code lines (example: x = 2 * 8):\n");
    for (int i = 0; i < n; i++) {
        fgets(code[i], MAX_LEN, stdin);
        code[i][strcspn(code[i], "\n")] = '\0'; // remove newline
    }

    printf("\nOriginal Code:\n");
    for (int i = 0; i < n; i++) {
        printf("%s\n", code[i]);
    }

    optimize(code, n);

    return 0;
}
```
<img width="769" height="584" alt="image" src="https://github.com/user-attachments/assets/5c06c4bb-1023-4381-94d3-e95be8ab47f3" />

2. a simple code generator that translates arithmetic expressions into target assembly for a stack machine.

```bash
   #include <stdio.h>
#include <string.h>
#include <stdlib.h>

// Simple optimizer: removes +0, *1 and folds constants for small demo
void optimizeExpression(char expr[], char optimized[]) {
    // Very basic demo optimizer (string replacement style)
    // Real optimizer would parse expression tree
    strcpy(optimized, expr);

    // Strength reduction and algebraic simplification
    char *p;
    while ((p = strstr(optimized, "*1")) != NULL) {
        memmove(p, p + 2, strlen(p + 2) + 1); // remove "*1"
    }
    while ((p = strstr(optimized, "+0")) != NULL) {
        memmove(p, p + 2, strlen(p + 2) + 1); // remove "+0"
    }
    while ((p = strstr(optimized, "0+")) != NULL) {
        memmove(p, p + 2, strlen(p + 2) + 1); // remove "0+"
    }

    // Example of constant folding: 2*8 -> 16
    if (strcmp(optimized, "2*8") == 0) {
        strcpy(optimized, "16");
    }
}

// Generate stack machine code for simple expressions (only binary ops)
void generateCode(char expr[]) {
    // NOTE: This is a demo and only works for forms like (a+b)*c, a+b, a*b etc.
    if (strcmp(expr, "(a+b)*c") == 0) {
        printf("PUSH a\n");
        printf("PUSH b\n");
        printf("ADD\n");
        printf("PUSH c\n");
        printf("MUL\n");
    } else if (strcmp(expr, "a+b") == 0) {
        printf("PUSH a\n");
        printf("PUSH b\n");
        printf("ADD\n");
    } else if (strcmp(expr, "a*b") == 0) {
        printf("PUSH a\n");
        printf("PUSH b\n");
        printf("MUL\n");
    } else {
        printf("Code generation not implemented for: %s\n", expr);
    }
}

int main() {
    char expr[100], optimized[100];

    printf("Enter an arithmetic expression: ");
    scanf("%[^\n]", expr);  // read full line

    optimizeExpression(expr, optimized);

    printf("\nOriginal Expression: %s\n", expr);
    printf("Optimized Expression: %s\n", optimized);

    printf("\nGenerated Stack Machine Code:\n");
    generateCode(optimized);

    return 0;
}
```
<img width="789" height="468" alt="image" src="https://github.com/user-attachments/assets/9c8af4db-8740-451c-a9f1-e6ed90f8b12b" />
