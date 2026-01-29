# Cricket Scorecard Management System (C)

This is a **terminal-based cricket match simulation program written in C**.
The project is developed as part of a **Data Structures** course to demonstrate the practical use of **stack, queue, and linked list** concepts.

The program simulates a limited-overs cricket match with real-time scoring, player statistics, fall of wickets, and match results.

---

## Features

* Two-team cricket match simulation
* User-defined team names and players
* Configurable number of overs
* Ball-by-ball score input
* Supports runs, wides, no-balls, and wickets
* Correct strike rotation
* Required run rate calculation during chase
* Fall of wickets display
* Batting scorecards for both teams
* Match result declaration

---

## Data Structures Used

### Linked List

* Used to store overs and balls dynamically.
* Each over contains a linked list of balls.

### Stack

* Used to store fall of wickets.
* Each dismissed batsman is pushed onto the stack.

### Queue

* Used to manage the batting order.
* Ensures the next batsman comes in correct sequence after a wicket.

---

## Program Flow

1. Enter team names and number of overs
2. Enter player names for both teams
3. Toss is conducted randomly
4. Toss winner chooses to bat or bowl
5. First innings is played
6. Target is set
7. Second innings is played
8. Match result is displayed
9. Scorecards are printed

---

## Ball Input Rules

| Input | Description                       |
| ----- | --------------------------------- |
| 0–6   | Runs scored                       |
| w     | Wide (1 run, ball not counted)    |
| n     | No-ball (1 run, ball not counted) |
| o     | Wicket                            |

---

## Compilation and Execution

Compile the program using:

```bash
gcc cricket_scorecard.c -o cricket
```

Run the program using:

```bash
./cricket
```

---

## File Structure

```
cricket_scorecard.c
README.md
```

---

## Limitations

* Bowling statistics are not implemented
* Extras like byes and leg-byes are not separated
* Input validation is minimal
* Only one match can be played per execution

---

## Conclusion

This project demonstrates how core data structures such as stack, queue, and linked list can be applied to a real-world problem like cricket score management.

---

## Author

Akhilesh

---

If you want it to sound **even more “average student”**, I can tone it down further.
If you want it to sound **exam-perfect**, I can adjust it for that too.
