# What method you use.
We use the Negamax algorithm to search the decision tree and employ the alpha-beta algorithm for pruning. Negamax algorithm is a variant form of minimax algorithm, which simplifies the search process, making the code more concise and easier to understand.
# Any unique aspects to it that you have developed.
To limit the search breadth and reduce computation time, a heuristic rule is employed to assign values to each move. The moves are then sorted based on these values. Subsequent searches only expand the first n moves in the set of moves.
# Any literature which has informed your decision process.
Rijswijck, J.V. (2000). Are Bees Better than Fruitflies? _Canadian Conference on AI_.
# What have you up to now achieved including evidence for the quality (e.g. performance).
Each decision-making step can be completed within approximately 15 seconds. Most of the time, the algorithm tends to choose the optimal or suboptimal solution for the current position. When playing as the red side, victory can be achieved in a very short time. However, when playing as the blue side, it may require slightly more time.
# Any experiments used to inform decisions, or for evaluation.
