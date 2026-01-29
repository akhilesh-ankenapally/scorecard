#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_PLAYERS 11
#define MAX_OVERS 20
#define MAX_QUEUE_SIZE 20
#define MAX_STACK_SIZE 10

// player statistics
typedef struct Player {
    char name[50];
    int runs;
    int balls;
    int isOut;
    int fours;
    int sixes;
} Player;
// team statistics
typedef struct Team {
    char name[50];
    Player players[MAX_PLAYERS];
    int totalRuns;
    int wickets;
} Team;
//balls in an over
typedef struct BallNode {
    char outcome[5];
    struct BallNode* next;
} BallNode;
// overs in an innings
typedef struct Over {
    char bowler[50];
    BallNode* balls;
    struct Over* next;
} Over;

typedef struct Stack {
    char names[MAX_STACK_SIZE][50];
    int top;
} Stack;

typedef struct Queue {
    char names[MAX_QUEUE_SIZE][50];
    int front, rear;
} Queue;

Team teamA, teamB;
Over* inningsHead = NULL;
int totalOvers;
int legalBalls = 0;
int currentOver = 0;
Stack wicketStack;
Queue battingQueue;
//fall of wickets
void initStack(Stack* s) { s->top = -1; }
void push(Stack* s, char* name) { if (s->top < MAX_STACK_SIZE - 1) strcpy(s->names[++s->top], name); }
void printWicketStack(Stack* s) {
    printf("Fall of Wickets:\n");
    for (int i = 0; i <= s->top; i++)
        printf("%d. %s\n", i + 1, s->names[i]);
}
//batting lineup
void initQueue(Queue* q) { q->front = q->rear = 0; }
void enqueue(Queue* q, char* name) {
    if ((q->rear + 1) % MAX_QUEUE_SIZE != q->front) {
        strcpy(q->names[q->rear], name);
        q->rear = (q->rear + 1) % MAX_QUEUE_SIZE;
    }
}

char* dequeue(Queue* q) {
    if (q->front == q->rear) return NULL;
    static char name[50];
    strcpy(name, q->names[q->front]);
    q->front = (q->front + 1) % MAX_QUEUE_SIZE;
    return name;
}

void insertBall(Over* over, char* outcome) {
    BallNode* newBall = (BallNode*)malloc(sizeof(BallNode));
    strcpy(newBall->outcome, outcome);
    newBall->next = NULL;
    if (over->balls == NULL) over->balls = newBall;
    else {
        BallNode* temp = over->balls;
        while (temp->next != NULL) temp = temp->next;
        temp->next = newBall;
    }
}

Over* addOver(char* bowlerName) {
    Over* newOver = (Over*)malloc(sizeof(Over));
    strcpy(newOver->bowler, bowlerName);
    newOver->balls = NULL;
    newOver->next = NULL;
    if (inningsHead == NULL) inningsHead = newOver;
    else {
        Over* temp = inningsHead;
        while (temp->next != NULL) temp = temp->next;
        temp->next = newOver;
    }
    return newOver;
}

float calculateRR(int runs, int balls) {
    return balls == 0 ? 0.0 : (runs * 6.0f / balls);
}

void printOverStatus(Over* over, Team* batting, Player* striker, Player* nonStriker) {
    printf("\n--- Over %d ---\n\n", currentOver + 1);
    printf("Bowler: %s | Score: %d/%d in %d.%d overs\n", over->bowler, batting->totalRuns, batting->wickets, legalBalls / 6, legalBalls % 6);
    printf("Striker: %s - %d(%d)\n", striker->name, striker->runs, striker->balls);
    printf("Non-Striker: %s - %d(%d)\n", nonStriker->name, nonStriker->runs, nonStriker->balls);
    printf("Run Rate: %.2f\n", calculateRR(batting->totalRuns, legalBalls));
}

void printScorecard(Team* team) {
    printf("\n\nScorecard for %s:\n", team->name);
    printf("%-15s %-6s %-6s %-6s %-6s %-10s\n", "Batsman", "R", "B", "4s", "6s", "SR");
    for (int i = 0; i < MAX_PLAYERS; i++) {
        if (team->players[i].balls > 0 || team->players[i].isOut) {
            float sr = team->players[i].balls == 0 ? 0.0 : ((float)team->players[i].runs / team->players[i].balls) * 100;
            printf("%-15s %-6d %-6d %-6d %-6d %-10.2f\n", team->players[i].name, team->players[i].runs, team->players[i].balls, team->players[i].fours, team->players[i].sixes, sr);
        }
    }
    printf("\nTotal: %d/%d\n", team->totalRuns, team->wickets);
}

int playInnings(Team* batting, Team* bowling, int target) {
    inningsHead = NULL;
    char strikerName[50], nonStrikerName[50], bowlerName[50];
    Player *striker = NULL, *nonStriker = NULL;
    int totalBalls = totalOvers * 6;
    int targetChase = target > 0;

    legalBalls = 0;
    currentOver = 0;

    initStack(&wicketStack);
    initQueue(&battingQueue);

    printf("\nSelect opening batsmen from %s:\n", batting->name);
    printf("Enter name of opening batsman 1: ");
    scanf("%s", strikerName);
    printf("Enter name of opening batsman 2: ");
    scanf("%s", nonStrikerName);

    for (int i = 0; i < MAX_PLAYERS; i++) {
        if (strcmp(batting->players[i].name, strikerName) == 0)
            striker = &batting->players[i];
        else if (strcmp(batting->players[i].name, nonStrikerName) == 0)
            nonStriker = &batting->players[i];
        else
            enqueue(&battingQueue, batting->players[i].name);
    }

    while (legalBalls < totalBalls && batting->wickets < 10) {
        printf("\nEnter bowler name for Over %d (%s): ", currentOver + 1, bowling->name);
        scanf("%s", bowlerName);
        Over* over = addOver(bowlerName);
        int ballsThisOver = 0;

        while (ballsThisOver < 6) {
            printOverStatus(over, batting, striker, nonStriker);
            if (targetChase) {
                int runsNeeded = target - batting->totalRuns;
                int ballsLeft = totalBalls - legalBalls;
                printf("Required RR: %.2f\n", calculateRR(runsNeeded, ballsLeft));
            }

            printf("Ball outcome (0-6, w=wide, n=no ball, o=wicket): ");
            char outcome[5];
            scanf("%s", outcome);
            insertBall(over, outcome);

            if (strcmp(outcome, "w") == 0 || strcmp(outcome, "n") == 0) {
                batting->totalRuns++;
                continue;
            }

            if (strcmp(outcome, "o") == 0) {
                batting->wickets++;
                striker->balls++;
                striker->isOut = 1;
                push(&wicketStack, striker->name);

                if (batting->wickets >= 10)
                    break;

                char* nextName = dequeue(&battingQueue);
                if (nextName == NULL) break;
                for (int i = 0; i < MAX_PLAYERS; i++) {
                    if (strcmp(batting->players[i].name, nextName) == 0) {
                        striker = &batting->players[i];
                        break;
                    }
                }

                ballsThisOver++;
                legalBalls++;
                continue;
            }

            int runs = atoi(outcome);
            striker->runs += runs;
            striker->balls++;
            if (runs == 4) striker->fours++;
            if (runs == 6) striker->sixes++;
            batting->totalRuns += runs;

            if (runs % 2 == 1) {
                Player* temp = striker;
                striker = nonStriker;
                nonStriker = temp;
            }

            ballsThisOver++;
            legalBalls++;

            if (targetChase && batting->totalRuns >= target)
                goto endInnings;
        }

        Player* temp = striker;
        striker = nonStriker;
        nonStriker = temp;

        currentOver++;
    }

endInnings:
    printf("\nInnings Over! Final Score: %d/%d\n", batting->totalRuns, batting->wickets);
    printWicketStack(&wicketStack);
    return legalBalls;
}

int main() {
    srand(time(0));

    printf("Enter Team A name: ");
    scanf("%s", teamA.name);
    printf("Enter Team B name: ");
    scanf("%s", teamB.name);

    printf("Enter number of overs: ");
    scanf("%d", &totalOvers);

    printf("Enter 11 player names for %s:\n", teamA.name);
    for (int i = 0; i < MAX_PLAYERS; i++) {
        printf("Player %d: ", i + 1);
        scanf("%s", teamA.players[i].name);
        teamA.players[i].runs = teamA.players[i].balls = teamA.players[i].isOut = teamA.players[i].fours = teamA.players[i].sixes = 0;
    }

    printf("Enter 11 player names for %s:\n", teamB.name);
    for (int i = 0; i < MAX_PLAYERS; i++) {
        printf("Player %d: ", i + 1);
        scanf("%s", teamB.players[i].name);
        teamB.players[i].runs = teamB.players[i].balls = teamB.players[i].isOut = teamB.players[i].fours = teamB.players[i].sixes = 0;
    }

    int tossWinner = rand() % 2;
    Team* tossTeam = tossWinner == 0 ? &teamA : &teamB;
    Team* otherTeam = tossWinner == 0 ? &teamB : &teamA;

    printf("\n%s won the toss!\n", tossTeam->name);
    char decision[5];
    printf("Choose to bat or bowl (bat/bowl): ");
    scanf("%s", decision);

    Team *batFirst, *bowlFirst, *batSecond, *bowlSecond;

    if (strcmp(decision, "bat") == 0) {
        batFirst = tossTeam;
        bowlFirst = otherTeam;
        batSecond = otherTeam;
        bowlSecond = tossTeam;
    } else {
        bowlFirst = tossTeam;
        batFirst = otherTeam;
        batSecond = tossTeam;
        bowlSecond = otherTeam;
    }

    printf("\n--- First Innings: %s Batting ---\n", batFirst->name);
    playInnings(batFirst, bowlFirst, 0);

    printf("\n--- Second Innings: %s Batting ---\n", batSecond->name);
    playInnings(batSecond, bowlSecond, batFirst->totalRuns);

    printf("\n--- Match Result ---\n");
    if (batSecond->totalRuns > batFirst->totalRuns)
        printf("%s won by %d wickets\n", batSecond->name, 10 - batSecond->wickets);
    else if (batSecond->totalRuns < batFirst->totalRuns)
        printf("%s won by %d runs\n", batFirst->name, batFirst->totalRuns - batSecond->totalRuns);
    else
        printf("Match Tied!\n");

    printScorecard(batFirst);
    printScorecard(batSecond);

    return 0;
}

// sample output:
 
// Enter Team A name: a
// Enter Team B name: b
// Enter number of overs: 2
// Enter 11 player names for a:
// Player 1: a1
// Player 2: a2
// Player 3: a3
// Player 4: a4
// Player 5: a5
// Player 6: a6
// Player 7: a7
// Player 8: a8
// Player 9: a9
// Player 10: a10
// Player 11: a11
// Enter 11 player names for b:
// Player 1: b1
// Player 2: b2
// Player 3: b3
// Player 4: b4
// Player 5: b5
// Player 6: b6
// Player 7: b7
// Player 8: b8
// Player 9: b9
// Player 10: b10    
// Player 11: b11

// b won the toss!
// Choose to bat or bowl (bat/bowl): bat

// --- First Innings: b Batting ---

// Select opening batsmen from b:
// Enter name of opening batsman 1: b1
// Enter name of opening batsman 2: b2

// Enter bowler name for Over 1 (a): a10

// --- Over 1 ---

// Bowler: a10 | Score: 0/0 in 0.0 overs
// Striker: b1 - 0(0)
// Non-Striker: b2 - 0(0)
// Run Rate: 0.00
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 2

// --- Over 1 ---

// Bowler: a10 | Score: 2/0 in 0.1 overs
// Striker: b1 - 2(1)
// Non-Striker: b2 - 0(0)
// Run Rate: 12.00
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 2

// --- Over 1 ---

// Bowler: a10 | Score: 4/0 in 0.2 overs
// Striker: b1 - 4(2)
// Non-Striker: b2 - 0(0)
// Run Rate: 12.00
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 0

// --- Over 1 ---

// Bowler: a10 | Score: 4/0 in 0.3 overs
// Striker: b1 - 4(3)
// Non-Striker: b2 - 0(0)
// Run Rate: 8.00
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 0

// --- Over 1 ---

// Bowler: a10 | Score: 4/0 in 0.4 overs
// Striker: b1 - 4(4)
// Non-Striker: b2 - 0(0)
// Run Rate: 6.00
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): w

// --- Over 1 ---

// Bowler: a10 | Score: 5/0 in 0.4 overs
// Striker: b1 - 4(4)
// Non-Striker: b2 - 0(0)
// Run Rate: 7.50
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 6

// --- Over 1 ---

// Bowler: a10 | Score: 11/0 in 0.5 overs
// Striker: b1 - 10(5)
// Non-Striker: b2 - 0(0)
// Run Rate: 13.20
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 1

// Enter bowler name for Over 2 (a): a11

// --- Over 2 ---

// Bowler: a11 | Score: 12/0 in 1.0 overs
// Striker: b1 - 11(6)
// Non-Striker: b2 - 0(0)
// Run Rate: 12.00
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 1

// --- Over 2 ---

// Bowler: a11 | Score: 13/0 in 1.1 overs
// Striker: b2 - 0(0)
// Non-Striker: b1 - 12(7)
// Run Rate: 11.14
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 0

// --- Over 2 ---

// Bowler: a11 | Score: 13/0 in 1.2 overs
// Striker: b2 - 0(1)
// Non-Striker: b1 - 12(7)
// Run Rate: 9.75
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 0

// --- Over 2 ---

// Bowler: a11 | Score: 13/0 in 1.3 overs
// Striker: b2 - 0(2)
// Non-Striker: b1 - 12(7)
// Run Rate: 8.67
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 0

// --- Over 2 ---

// Bowler: a11 | Score: 13/0 in 1.4 overs
// Striker: b2 - 0(3)
// Non-Striker: b1 - 12(7)
// Run Rate: 7.80
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 1

// --- Over 2 ---

// Bowler: a11 | Score: 14/0 in 1.5 overs
// Striker: b1 - 12(7)
// Non-Striker: b2 - 1(4)
// Run Rate: 7.64
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 2

// Innings Over! Final Score: 16/0
// Fall of Wickets:

// --- Second Innings: a Batting ---

// Select opening batsmen from a:
// Enter name of opening batsman 1: a1
// Enter name of opening batsman 2: a2

// Enter bowler name for Over 1 (b): b8

// --- Over 1 ---

// Bowler: b8 | Score: 0/0 in 0.0 overs
// Striker: a1 - 0(0)
// Non-Striker: a2 - 0(0)
// Run Rate: 0.00
// Required RR: 8.00
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 6

// --- Over 1 ---

// Bowler: b8 | Score: 6/0 in 0.1 overs
// Striker: a1 - 6(1)
// Non-Striker: a2 - 0(0)
// Run Rate: 36.00
// Required RR: 5.45
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 6

// --- Over 1 ---

// Bowler: b8 | Score: 12/0 in 0.2 overs
// Striker: a1 - 12(2)
// Non-Striker: a2 - 0(0)
// Run Rate: 36.00
// Required RR: 2.40
// Ball outcome (0-6, w=wide, n=no ball, o=wicket): 6

// Innings Over! Final Score: 18/0
// Fall of Wickets:

// --- Match Result ---
// a won by 10 wickets


// Scorecard for b:
// Batsman         R      B      4s     6s     SR
// b1              14     8      0      1      175.00
// b2              1      4      0      0      25.00

// Total: 16/0


// Scorecard for a:
// Batsman         R      B      4s     6s     SR
// a1              18     3      0      3      600.00

// Total: 18/0
