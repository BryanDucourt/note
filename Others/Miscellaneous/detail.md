# What method you use.
We use the Negamax algorithm to search the decision tree and employ the alpha-beta algorithm for pruning. Negamax algorithm is a variant form of minimax algorithm, which simplifies the search process, making the code more concise and easier to understand.
# Any unique aspects to it that you have developed.
To limit the search breadth and reduce computation time, a heuristic rule is employed to assign values to each move. The moves are then sorted based on these values. Subsequent searches only expand the first n moves in the set of moves.
# Any literature which has informed your decision process.
Rijswijck, J.V. (2000). Are Bees Better than Fruitflies? _Canadian Conference on AI_.
# What have you up to now achieved including evidence for the quality (e.g. performance).
Each decision-making step can be completed within approximately 15 seconds. Most of the time, the algorithm tends to choose the optimal or suboptimal solution for the current position. When playing as the red side, victory can be achieved in a very short time. However, when playing as the blue side, it may require slightly more time.
# Any experiments used to inform decisions, or for evaluation.
Multiple experiments were conducted during the process of adjusting search depth and breadth. Optimization adjustments were made to parameters controlling search depth and breadth based on the time spent on decision-making and the quality of decisions. Addressing issues related to suboptimal choices during the decision-making process, the evaluation algorithm underwent targeted optimizations. Different weights were assigned to components in the evaluation formula based on the player's role, and a parameter measuring the distance from a position to the center position was introduced.
# Quantitative measures of performance (depth of search, number of play- outs per move, training time, etc.).
The search depth is limited to 4, and the number of simulations per move is restricted to 5.
# What you expect to have accomplished by submission.
At the time of submission, our goal is for the agent to achieve victory within a reasonable time frame in the majority of games and to minimize the occurrence of unreasonable moves.
#  Each member of the group should say something about the content, for example, their contribution to the project.
pass

# Secret sauce
Introducing heuristic rules to assess and sort moves.By adjusting the parameters of the evaluation formula, each role could primarily focus on its own potential value, while also taking the center factor and the opponent's potential value into consideration.