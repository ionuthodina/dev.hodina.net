## How to create a gold league bot on CodinGame

### Intro
The CodinGame platform has a dedicated area for programming game bots. First, you have to choose the game to play. Then it provides the code template for the initial context and the moves of the opponent player. For each turn, you should calculate and provide your best moves in less than 100ms. The opponent can be the league AI implementation or another developer implementation. This way, your bot can compete with other users' bots and advance through leagues: Wood, Bronze, Silver, Gold, and Legend. By competing against others, you can be quite sure how robust your algorithm is.

You can code in any of the 26 programming languages available. The cool part is that every game comes with built-in visual simulation so it's easier to debug and notice anomalies in the algorithm.

### Ultimate Tic Tac Toe
Before this, I didn't know about the extended version of the well-known game. Well, I would say it's on another level of complexity to just play it, not to mention to code it.

The rules are well explained [here](https://www.thegamegal.com/2018/09/01/ultimate-tic-tac-toe/). 

It is not fair to paste the entire source code. Nor I will explain in detail the algorithms when there is already enough information. But I will mention in the bibliography the best sources that helped me. Additionally, I will provide all the details that made a difference in the final implementation. And other tries that failed.

### The Bronze league
You probably have already implemented the classic game a few times. It's not way harder for the extended version. You can keep it simple and implement it however you want. I started to do it in an object-oriented fashion and had to manage the following entities: MiniBoard, MainBoard, Player, and Game. State of a MiniBoard can be an array of 9 int/char entries and MainBoard can be an array of 9 MiniBoard items. Obvious, right?!

Logic can be straightforward: prioritize 3 in a row winning/defending a MiniBoard, or choose one of the best moves: center and corners. Hope for the best, and your bot can advance to the Bronze league with minimum effort.

### The challenge
No matter how well you play this game, you cannot code all possible (best) moves. Compared with a basic game, the Ultimate version has just too many possible states of boards and it's not obvious why you should not win a MiniBoard, just to have a better chance to win the game 5 moves later.

### Minimax algorithm?
Can be a good option if you are comfortable trusting some heuristics methods that would rate the next possible 5-10 moves. Then it would choose the best one: minimize opponent chances and maximize your chances. The problem is that you should already know some strategies and tricks to correctly evaluate the state of a board. Then should be easy to code evaluation heuristics.

### Monte Carlo Tree Search
[MCTS](https://en.wikipedia.org/wiki/Monte_Carlo_tree_search) uses random game playouts to the end, instead of the usual static evaluation function. The expected-outcome model *is shown to be precise, accurate, easily estimable, efficiently calculable, and domain-independent.*

Simply put: let the algorithm simulate a lot of games, and over time it will choose better and better moves. The big advantage is that you don't have to be a domain expert, to know how to win the game. You only need to code the rules and the expected outcome.

Following different articles, you will get to have an implementation like this:

```
static int WinScore = 10;

public Point FindNextMove(MainBoard mainBoard)
{
    Node rootNode = new Node(mainBoard);
    var end = DateTime.Now.AddMilliseconds(90);

    while (DateTime.Now < end)
    {
        ExploreBestNodes(rootNode);
    }

    Node winnerNode = rootNode.GetChildWithMaxScore();
    return winnerNode.State.Board.LastMove;
}

public void ExploreBestNodes(Node rootNode)
{
    Node promisingNode = SelectPromisingNode(rootNode);
    if (promisingNode.State.Board.Status == MainBoardStatus.Playable)
    {
        ExpandNode(promisingNode);
    }

    Node nodeToExplore = promisingNode;
    if (promisingNode.Childs.Count > 0)
    {
        nodeToExplore = promisingNode.GetRandomChildNode();
    }

    MainBoardStatus playoutResult = SimulateRandomPlayout(nodeToExplore);
    BackPropogation(nodeToExplore, playoutResult);
}

private void BackPropogation(Node nodeToExplore, MainBoardStatus boardStatus)
{
    Node tempNode = nodeToExplore;
    while (tempNode != null)
    {
        tempNode.State.VisitCount++;
        if (boardStatus == MainBoardStatus.WonByP1)
        {
            tempNode.State.Score += WinScore;
        }

        tempNode = tempNode.Parent;
    }
}

private MainBoardStatus SimulateRandomPlayout(Node node)
{
    Node tempNode = new Node(node);
    State tempState = tempNode.State;
    var boardStatus = tempState.Board.Status;

    if (boardStatus == MainBoardStatus.WonByP2)
    {
        tempNode.Parent.State.Score = int.MinValue;
        return boardStatus;
    }

    int rounds = 100;
    while (boardStatus == MainBoardStatus.Playable && rounds> 0)
    {
        tempState.RandomPlay();
        boardStatus = tempState.Board.Status;
        rounds--;
    }

    return boardStatus;
}

private static double GetUctValue(int totalVisits, double score, int visits)
{
    if (visits == 0)
    {
         return int.MaxValue;
    }
    return (score / visits) + 1.41 * Math.Sqrt(Math.Log(totalVisits) / visits);
}
``` 

### The Silver league
You should advance to the silver league. MCTS and UCT (Upper Confidence Trees) work like magic! From now on, your bot can become smart.

### Precalculated moves before the first round
Only for the first round, there is a limit of 1000ms instead of 100ms. That means more available time for simulations, therefore better moves.
I tried to calculate and cache those moves at the start to have an advantage. But after a lot of tests, didn't seem to matter too much. I suppose it's because the impact of the first 3-5 moves is minimal considering there are 50 more. I did a rollback.

### Optimize, optimize, optimize
Using MCTS means that you should squeeze in as much computational time as possible. You want to have a lot of simulated playouts for UCT value to converge to better results. Don't underestimate 10% improvement as, over many rounds, your bot would play exponentially better.

Having more simulations does help. But it's way better to let the bot play games with higher chances of winning. Here are a few tips and tricks:
- When you start the game, play a hardcoded best position. I chose (4, 4), centered MiniBoard, left-up corner square.
- When a MiniBoard is (almost) empty choose to play first the best positions: center and corners.
- When you have to choose the current MiniBoard(the next board is full or already won), evaluate states for all others.
- Pay attention to custom logic. Takes more time, reduces simulations, and may introduce a lot of bias. It happened to me, and I had to delete most of the custom code. Keep it simple and add *improvements* only after being individually tested. Don't forget that the power of MCTS is in random playouts.

### Test different parameters
To get the best results, parameters need to be correctly adjusted.
- Find the right balance of Exploitation vs Exploration weights of UCT value. I tested WinScore to be in the range [1, 100] and the exploration constant from sqrt(2) to 2. For me worked best with WinScore=6 and c=sqrt(2).
- Most of the games finish in 50-60 moves. There is a waste of time to let the bot simulate way longer games. Remember that the bot must win as fast as possible, not play every available square.
```
int rounds = 70 - node.State.Board.CurrentRound;
``` 
- Teach the bot to work smarter, not harder. Give a small penalty if the game is won with more rounds.

```
double score = WinScore;
-----------
tempNode.State.Score += score;
score *= .99;
```

### Use bits for game state
Probably you would think to represent your board state like an array of 9 integers: 0 - unassigned, 1 - won by X, 2 - won by O. But, instead of that, it can be used only 1 number! A C# integer has 32 bits. So, each bit can record 2 states: 0 - unassigned, 1 - won. You can encode any kind of information you want.

This is an old trick to get the best performance in terms of computing and memory management. Bitwise operators are very fast. Additionally, loops and checks are dramatically improved. I chose to have:
- First bit to be 0 if board is still playable. Otherwise to be set to 1.
- Next 9 bits for X player and next 9 bits for O player. So if X won the board by having second row, it would look like:
```
int board = 0b_1_000111000_000000000;
```
So how to test if a player won main diagonal?
```
int currentPlayer = 0; // or 1
int winMask = 0b_100010001; // or loop through all winning masks
int playerAdjustedMask = winMask << currentPlayer * 9; // toggle bits mask shifting based on current player
var result = board & playerAdjustedMask;
return result == playerAdjustedMask; // magic!
```
How to test for a draw?
```
return ((board | (board >> 9)) & 511) == 511; // Check if positions of both players are set knowing that 511 is 111111111 in binary
```

### The Gold league
If you follow all the tips, you should arrive in Gold league. Here I stopped my journey as it took me way too many days. It was both annoying and fun. You know, standard programmer feelings on daily tasks. I learnt a lot of stuff and I'm very happy that I created a game bot that can actually beat me on this.

Did you create a similar bot? What other improvements did you make?
What did you learn?


### Bibliography
- [Monte-Carlo Tree Search](http://matthewdeakos.me/2018/03/10/monte-carlo-tree-search/)
- [Monte Carlo Tree Search for Tic-Tac-Toe Game | Baeldung](https://www.baeldung.com/java-monte-carlo-tree-search)
- [Minimax and Monte Carlo Tree Search - Philipp Muens](https://philippmuens.com/minimax-and-mcts)
- [Pitfalls and Solutions When using MCTS for Strategy and Tactical Games](http://www.gameaipro.com/GameAIPro3/GameAIPro3_Chapter28_Pitfalls_and_Solutions_When_Using_Monte_Carlo_Tree_Search_for_Strategy_and_Tactical_Games.pdf)
- [Algorithm: Winner in Tic-Tac-Toe](https://rclayton.silvrback.com/winner-in-tic-tac-toe)
- [Tic-Tac-Toe Bitboards](https://libfbp.blogspot.com/2017/05/tic-tac-toe-bitboards.html)
- [A Survey of Monte Carlo Tree Search Methods](http://www.incompleteideas.net/609%20dropbox/other%20readings%20and%20resources/MCTS-survey.pdf)
- [Transpositions and Move Groups in Monte Carlo Tree Search](https://www.csse.uwa.edu.au/cig08/Proceedings/papers/8057.pdf)