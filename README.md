# ChineseChess
Command line Chinese chess program which supports human-computer competition

Overview
	This document servered as a report of a command line Chinese chess program that supports human-computer competition. The program is implemented in Java and is cross-platform. Users can either executed in Windows, Machintosh or Linux operating systems. Make sure Java is pre-installed in running enviornments.
	Chapter 1 will discuss installaion procedure and the program usage. Chapter 2 will discuss searching methodology and methods or heuristics that improves searching time and memories. Chapter 3 will provide details about evaluation methodology. Chapter 4 will discuss the performance of the program, as well as heuristics applied to strive the balance between searching depth and time. Finally it will discuss some possible improvements of the program. 


Chapter 1: Installation and Usage:
	The program is already compiled in bin folder and you can directly execute the program under bin folder. However, if you need to recompile it for any reasons, please enter the following commands in your terminal/cmd/prompt:

			javac ConsoleGame.java

	If the program is compiled, class files will be generated in the same folder. You can execute the program by the following command:

			java ConsoleGame

	Once the program starts, messages and board will be displayed in terminal as follows:


	///////////////////////////////////////////////////////////////////////////////////////////////////
		1	2	3	4	5	6	7	8	9	
	A	(車)	(馬)	(象)	(士)	(將)	(士)	(象)	(馬)	(車)	
	B	.	.	.	x	x	x	.	.	.	
	C	.	(炮)	.	x	x	x	.	(炮)	.	
	D	(卒)	.	(卒)	.	(卒)	.	(卒)	.	(卒)	
	E	----	----	----	----	----	----	----	----	----	
	F	----	----	----	----	----	----	----	----	----	
	G	<兵>	.	<兵>	.	<兵>	.	<兵>	.	<兵>	
	H	.	<砲>	.	x	x	x	.	<砲>	.	
	I	.	.	.	x	x	x	.	.	.	
	J	<車>	<馬>	<相>	<仕>	<帥>	<仕>	<相>	<馬>	<車>	
	///////////////////////////////////////////////////////////////////////////////////////////////////

	Tex game board consists of 9*10 grids points and 32 pieces. Grid points with empty piece are represented by character '.', while points within fortress are indecated by 'x'. River, as well as middle positions, are represented by four consecutive characters '-'.
	For text representation of pieces, it is displayed by its Chinese name and enclosed braket. Red pieces are enclosed by '<', '>' , while black piece are enclosed by '(' and ')'. Following table shows the corresponding pieces representation and English name of each piece: 

	+-------+------+---------+--------+--------+------+--------+------+
	| Color | King | Advisor | Bishop | Knight | Rook | Cannon | Pawn | 
	+-------+------+---------+--------+--------+------+--------+------+
	| Black | (將)  | (士)   | (象)    | (馬)   | (車)  | (炮)   | (卒) | 
	| Red   | <帥>  | <仕>   | <相>    | <馬>   | <車>  | <砲>   | <兵> | 
	+-------+------+---------+--------+--------+------+--------+------+

	Letters and numbers at the border represents the x-y coordinates of each piece. For example, coordinate "A1" represents Rook (車) at the top left corner.
	To make a move, player can enter starting and ending coordinates in the format <letter><number>,<letter><number>. For example, "A1,A3" would be moving piece Pawn from A1 to A3.

	Also, follow shows some commands that player can uses the following commands during game:

	Commands:	
	capture: Display captured pieces
	display: Display chess board
	history: Display previous moves
	help:    Display avaliable options
	quit:    Quit game
	restart: Restart game
	swap:    Swap player pieces
	undo:    Undo move
	<move>:  A move with the format <letter><number>,<letter><number>

Chapter 2: Searching Methedology:

	The following shows a simplified version of minimax procedure with alpha-beta pruning that extracted from Search.java:
	///////////////////////////////////////////////////////////////////////////////////////////////////
	private int minimax(int depth,char currentColor, char rootColor, int alpha, int beta) {
        /* Return evaluation if reaching leaf node or any side won.*/
        if (depth == 0 || controller.getState(board) != 'x')
            return Evaluate.evaluate(board, ((rootColor==Piece.BLACK)?Piece.BLACK:Piece.RED));
        ArrayList<Node> moves = generateAllMoves(currentColor);
        	for (Node n : moves) {
        		//Update state
                Piece capturedPiece = board.move(n.piece, n.to);
                if (rootColor==currentColor) 
                	alpha = Math.max(alpha, minimax(depth - 1,swapColor(currentColor),rootColor , alpha, beta));
                else 
                	beta = Math.min(beta, minimax(depth - 1,swapColor(currentColor),rootColor , alpha, beta));
                
                //Restore state
                board.undo(board.pieces.get(n.piece), n.from, capturedPiece);

                /* Alpha-beta Pruning */
                if (alpha>=beta) break;
            }
        return rootColor==currentColor ? alpha : beta;
    } 
	///////////////////////////////////////////////////////////////////////////////////////////////////
	The value of alpha and beta stores the maximum and minimum value evaluated root state. Node that function swapColor() is applied to each depth, minimum and maximum will be computed alternately with depth, that will eventually returns a minimax value in the search tree.

	During implementation, I originally tried to return a Node instead of int. Excessive amounts of memeories are used in cloning chess board as well as whole set of pieces in each node, which result in memory overflow and unreasonly long searching time. In this modified version, no extra memories is allocated to chess board and chess piece, instead I would perform upadte and undo in the same board and same set of pieces. 
	
	After implementing minimax algorithm, implementing alpha-beta pruning would be relatively easy, by adding two arguments and a conditional statements in the function. As the for loop will be break, unnecessary node therefore will not be explored and hence treated as cut-off.
	
	For move generation, the function will perform shell sort before returning list of nodes. As the order of node will affect the alpha or beta value, the best situation is to find the "best" move at the earlier stage such that more unnecessary nodes will not be explored. For striving the goal, some heuristics will be applied. I assume that the piece strength are highly correlated to best move, in other words, by searhing pieces with high strength value, it is more likely to find the minimax move earlier, so the comparing function is simply using their piece strength from strength evaluation function in sorting algorithm. In usual case, rook, knight or cannon will be evaluated before pawns.
	
	At the root node, state of game board would be evaluted by an evaulate function. Details of the function will be discussed in later section.
	Noted that rootColor is passed to evaluator instead of current color. It is becuase the evaluated state must be consistant with search color, otherwise

Chapter 3: Evaluation
	Evaulation function treated as an indicator of reflecting the performance in current state. 
	The evaluation consists of t parts, including piece values, position values, mobility, threat and control values. The following source code extracted from Evaluate.java illustrates the function:

	///////////////////////////////////////////////////////////////////////////////////////////////////
    public static int evaluate(Board board, char player) {
    	
    	int[]strength = new int[2];
    	int[]material = new int[2];
    	int[]mobility = new int[2];
    	int[]threat = new int[2];
    	int[]control = new int[2];
        int redValue=0;
        int blackValue=0;
    	
        strength=strength(board);
        material=material(board);
        mobility=mobility(board);
        threat=threat(board);
        control=control(board);
        redValue=6*strength[RED_INDEX]+4*material[RED_INDEX]+4*threat[RED_INDEX]+1*mobility[RED_INDEX]+1*control[RED_INDEX];
        blackValue=6*strength[BLACK_INDEX]+4*material[BLACK_INDEX]+4*threat[BLACK_INDEX]+1*mobility[BLACK_INDEX]+1*control[BLACK_INDEX];
        
        return ((player==Piece.RED)?redValue-blackValue:blackValue-redValue);
        
    }
	///////////////////////////////////////////////////////////////////////////////////////////////////

	redValue and blackValue represents the linear combination of the 5 evaluted components in each color. Therefore evaluate function would return the difference of redValue and blackVale, determined by player color. If the value is positive given a certain player, it means that the player is in advantageous, in other words, the overall strength, material, threat, mobility and control is greater than opponent. Otherwise it is in disadvantageous. For preventing domination of certain evaluated component, normalizing coefficients are assigned to each component. For simplicity, coefficients are defined as constant and will not be changed. The meaning of each evaluated component will be discussed as belows: 

	1)Piece Strength

	The following table extracted from Evaluate.java shows the evaluted strength of each piece:

	+---------+---------+--------+--------+------+--------+------+
	|  King   | Advisor | Bishop | Knight | Rook | Cannon | Pawn |
	+---------+---------+--------+--------+------+--------+------+
	| 1000000 |     110 |    110 |    300 |  600 |    500 |   70 |
	+---------+---------+--------+--------+------+--------+------+

	The strength hence is computed by summing the value of existing pieces on the board. If a piece is captured, value will not be counted, so the overall strength is weaken. Noted that the value of king is significantly larger than other pieces, so that computer will search for the best way to keep king alive, or to capture opponents king.

	2)Position value

	Position evalutes the value of certain position given a position occupied by piece. Considering the following evalute table of king, and pawn from extracted Evaluate.java, the position value hence can be computed by locating position index of pieces. Noted that the table is placed according to red position. For black pieces, we would mirror vertically before evaluation.

	int cucvlKingPawnMidgameAttackless[][] = {
	  { 9,  9,  9, 11, 13, 11,  9,  9,  9},
	  {19, 24, 34, 42, 44, 42, 34, 24, 19},
	  {19, 24, 32, 37, 37, 37, 32, 24, 19},
	  {19, 23, 27, 29, 30, 29, 27, 23, 19},
	  {14, 18, 20, 27, 29, 27, 20, 18, 14},
	  { 7,  0, 13,  0, 16,  0, 13,  0,  7},
	  { 7,  0,  7,  0, 15,  0,  7,  0,  7},
	  { 0,  0,  0,  1,  1,  1,  0,  0,  0},
	  { 0,  0,  0,  2,  2,  2,  0,  0,  0},
	  { 0,  0,  0, 11, 15, 11,  0,  0,  0}
	};

	The above table shows that value of pawn increases significantly when approching to enemy side. By adopting similiar table to other piece types, computer tends to move towards those points that have much significant position values. It is useful especially in the beginning of the game. As search depth is limited, computer can only relies on the position value for making decisions.

	3)Mobility
	Restricting enemy pieces would greatly improves player's attacking power. If a piece could not move freely under the rule of Chinese chess, its attacking or defending power is restricted. In other word, the more flexible the piece is, the more attacking or defending power it has. In the implementation, value of mobility measured by weighted sum of pieces and its number of possible moves. For simplicity, only moves and weighting of knight, rook, cannon and pawn will be considerd. The weighting of each pieces is shown in the following table:

	+--------+------+--------+------+
	| Knight | Rook | Cannon | Pawn |
	+--------+------+--------+------+
	|    13  |  7   |    7   |   15 |
	+--------+------+--------+------+

	4)Threat
	Threat refers to somes special patterns or combination of pieces in attacking opponent. For simplicity, we consider pieces to be threatening only when the piece crosses the river.

	5)Control
	Similiar to mobility, but control of a piece is measured by the position values that the piece can be captured in the next move, instead of possible moves. The more control positions, the more powerful the move it is.

	To summarize, evaluate function is composed by serval characteristics given by each pieces. By adjusting weighting of each evaluated component, the behaviour of the computer will be different. For example, giving unreasonly high weighting on position value would gives 

Chapter 4: Performance

	At the beginning of game, the program explored approximately  60000 nodes in around 1 second with depth 3. As no open or ending database implemented in the program, the searching time rapidly increased at the beginning stage and would cause lot more time for deeper look forward steps. Since the search depth is not sufficient enough, that the program behaves as a novice player in some cases, rook or knight can be easily captured in the beginning stage. The situation is improved by adopting a more comprehensive evaulating function, however, it is highly relays on evaluating function, instead of look forward steps.

	To strive the balance between searching time and depth, the depth increases progressively with the number of remaining pieces on game board.
	By setting a limited searching depth of 9, the relation between depth and number of pieces can be expressed in the following equations:

			depth = Math.min(9,-0.11*board.pieces.size()+7.3);

	In other words, when there are less then 30 pieces, searching depth will be increased to 4, with around 800000 evaluated nodes in each move. In such case, searching time will be increased to around 6 seconds in each move. It is possible that uses more that 10 seconds to explored over 1500000 nodes. Meanwhile, moves generated by computer becomes more reasonable and aggressive.

	When closed to the end, branching factors would decrease graudually, such that increasing search depth will not causes significant effects on searching time. In such case, the depth usually adjusted to 5. 

	To summarize, searching depth increases from 3 to 5 in the game, according to number of existing pieces on the board. Regarding the problems of insufficient searching depth encounted in the beginning stage, database of opening pattern can be introduced such that computer can be lookup the pattern from database directly instead of searching.
