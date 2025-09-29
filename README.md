# TOC
1. Program to convert a given regular expression into its equivalent DFA

```bash
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX 50

// ---------- Data Structures ----------
typedef struct {
    int states[MAX];   // NFA states in the DFA state
    int n;             // Number of NFA states inside
    int isFinal;       // 1 if this DFA state is final
} DFAState;

typedef struct {
    int nfaStart;
    int nfaEnd;
} NFAEdge;

// ---------- Global Variables ----------
int nfa[10][3];  // nfa[state][symbol], symbol: 0=a, 1=b, 2=ε
int nfaStates = 0;
DFAState dfa[MAX];
int dfaCount = 0;
char inputSymbols[2] = {'a', 'b'};

// ---------- Utility Functions ----------
int isPresent(int arr[], int n, int x) {
    for (int i = 0; i < n; i++)
        if (arr[i] == x) return 1;
    return 0;
}

void addState(int arr[], int *n, int x) {
    if (!isPresent(arr, *n, x))
        arr[(*n)++] = x;
}

void epsilonClosure(int start, int closure[], int *n) {
    addState(closure, n, start);
    for (int i = 0; i < *n; i++) {
        int s = closure[i];
        if (nfa[s][2] != -1 && !isPresent(closure, *n, nfa[s][2])) {
            addState(closure, n, nfa[s][2]);
        }
    }
}

void moveFunc(int from[], int n, int symbol, int result[], int *rn) {
    for (int i = 0; i < n; i++) {
        int st = from[i];
        if (nfa[st][symbol] != -1) {
            addState(result, rn, nfa[st][symbol]);
        }
    }
}

// ---------- NFA for (a|b)*abb ----------
// States: 0(start),1,2,3,4
// Accepting state: 4
// Transitions (0:(a,b)* loop)
void buildNFA() {
    // Initialize
    for (int i = 0; i < 10; i++)
        for (int j = 0; j < 3; j++)
            nfa[i][j] = -1;

    // (a|b)* : loop on state 0
    nfa[0][0] = 0;  // a
    nfa[0][1] = 0;  // b

    // then 'a'
    nfa[0][0] = 1; // a -> state1
    // but keep loops: we allow repetition
    // Actually we can nondeterministically go
    // We'll model like: 0 -a-> 0 also
    // For simplicity: keep 0->0 also
    // But we focus on abb

    // Let’s create chain for abb
    nfa[0][0] = 1; // on 'a' can start abb
    nfa[0][0] = 0; // loop
    // It's tricky to hardcode. Instead build carefully.

    // Simpler explicit chain:
    // We'll treat (a|b)* as loops on 0, plus transitions toward abb
    nfa[0][0] = 0; // a loop
    nfa[0][1] = 0; // b loop
    // but to detect abb we add path:
    nfa[0][0] = 1; // if 'a' begin abb
    nfa[1][1] = 2; // after a, b
    nfa[2][1] = 3; // after ab, b
    nfaStates = 4; // states 0..3
}

// ---------- Subset Construction ----------
int compareStates(int a[], int na, int b[], int nb) {
    if (na != nb) return 0;
    for (int i = 0; i < na; i++)
        if (!isPresent(b, nb, a[i]))
            return 0;
    return 1;
}

int getDFAState(int states[], int n) {
    for (int i = 0; i < dfaCount; i++) {
        if (compareStates(dfa[i].states, dfa[i].n, states, n))
            return i;
    }
    return -1;
}

void subsetConstruction() {
    int startClosure[MAX], n=0;
    epsilonClosure(0, startClosure, &n);
    dfa[0].n = n;
    for (int i = 0; i < n; i++) dfa[0].states[i] = startClosure[i];
    dfa[0].isFinal = isPresent(dfa[0].states, dfa[0].n, 3);
    dfaCount = 1;

    int transitions[MAX][2];
    memset(transitions, -1, sizeof(transitions));

    for (int i = 0; i < dfaCount; i++) {
        for (int sym = 0; sym < 2; sym++) {
            int moveRes[MAX], nm = 0;
            moveFunc(dfa[i].states, dfa[i].n, sym, moveRes, &nm);

            int closure[MAX], nc = 0;
            for (int k = 0; k < nm; k++)
                epsilonClosure(moveRes[k], closure, &nc);

            if (nc == 0) continue;
            int idx = getDFAState(closure, nc);
            if (idx == -1) {
                idx = dfaCount;
                for (int k = 0; k < nc; k++)
                    dfa[idx].states[k] = closure[k];
                dfa[idx].n = nc;
                dfa[idx].isFinal = isPresent(closure, nc, 3);
                dfaCount++;
            }
            transitions[i][sym] = idx;
        }
    }

    // Print DFA Transition Table
    printf("\nEquivalent DFA Transition Table:\n");
    printf("-------------------------------------------------\n");
    printf("| State |   a   |   b   | Final |\n");
    printf("-------------------------------------------------\n");
    for (int i = 0; i < dfaCount; i++) {
        printf("|  %d   |", i);
        for (int sym = 0; sym < 2; sym++) {
            int moveRes[MAX], nm = 0;
            moveFunc(dfa[i].states, dfa[i].n, sym, moveRes, &nm);
            int closure[MAX], nc = 0;
            for (int k = 0; k < nm; k++)
                epsilonClosure(moveRes[k], closure, &nc);
            if (nc == 0)
                printf("  -  |");
            else
                printf("  %d  |", getDFAState(closure, nc));
        }
        printf("   %s   |\n", dfa[i].isFinal ? "Yes" : "No");
    }
    printf("-------------------------------------------------\n");
}

// ---------- Main ----------
int main() {
    buildNFA();
    subsetConstruction();
    return 0;
}
```
<img width="804" height="427" alt="image" src="https://github.com/user-attachments/assets/6261f257-bd06-4a45-aa91-af0d32f8f07d" />

2. Program that converts the following RE into DFA and validates strings

   ```bash
   #include <stdio.h>
#include <string.h>

// DFA states
// q0 = 0, q1 = 1, q2 = 2
int transition(int state, char symbol) {
    switch (state) {
        case 0: // q0
            if (symbol == '0') return 1; // go to q1
            else if (symbol == '1') return 0; // stay in q0
            break;
        case 1: // q1 (last was 0)
            if (symbol == '0') return 1; // stay in q1
            else if (symbol == '1') return 2; // 0 followed by 1 -> q2
            break;
        case 2: // q2 (ends with 01)
            if (symbol == '0') return 1; // could start new pattern
            else if (symbol == '1') return 0; // back to q0
            break;
    }
    return -1; // invalid
}

int isAccepted(const char *input) {
    int state = 0; // start state
    for (int i = 0; i < (int)strlen(input); i++) {
        if (input[i] != '0' && input[i] != '1') return 0; // invalid char
        state = transition(state, input[i]);
    }
    // Final state is q2
    return (state == 2);
}

int main() {
    const char *testStrings[] = {"1101", "111", "0001"};
    int count = sizeof(testStrings) / sizeof(testStrings[0]);

    printf("Regular Expression: (0|1)*01\n");
    printf("Testing strings:\n\n");

    for (int i = 0; i < count; i++) {
        printf("String: %-6s --> %s\n",
               testStrings[i],
               isAccepted(testStrings[i]) ? "Accepted" : "Rejected");
    }

    return 0;
}
```
<img width="824" height="365" alt="image" src="https://github.com/user-attachments/assets/2f7faee3-2ba2-4bd2-bdf7-f2fa6a677508" />




