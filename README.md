# aimms-santa-2021
A simple AIMMS project to address the Santa 2021 Kaggle challenge.

## How to run
In order to run the project, you can use the MainExecution procedure or follow the 4 steps below:
1. Run Proc_GeneratePermutations - This generates all permutations and saves them in the permutations set.
2. Run Proc_CalculateDistances - This calculates distances between permutations. Any incompatible permutations will have a 999 distance.
3. Run the Proc_HeuristicAllocation - This will run an allocation heuristic splitting the problem into 3 (1 for each elf group)
4. Run the Proc_HeuristicRun - This will run the TSP with MLZ formulation for each elf group individually)
5. Run the Proc_ExportToKaggle - This will generate a csv file ready to submit to the Kaggle challenge

## Additional information
Some disclaimers and information on the implementation:
1. Due to complexity this implementation is not a global optimization model. I applied some heuristics, and I broke the model into smaller portions to make it feasible to run on my laptop.
2. I had some issues with the encoding of the files provided by Kaggle. You view the permutation generations procedure at Proc_GeneratePermutations and Proc_CalculateDistances.
    * With the encoding problems, each film string is represented by 2 characters. You’ll notice I use this information in certain Substring call, multiplying by 2 the size of the string.
3. The model strategy is:
    * A greedy algorithm allocates the film permutations to each individual elf group in a simple “minimum distance from previous permutation” approach. All forced permutations (as per described in the challenge) are included in this process as well. The process is finished when all elf groups have all forced permutations and also the free permutations have been picked up by 1 group. First permutation is randomly assigned to each group. You can find the algorithm in Proc_HeuristicAllocation.
    * Then we run individual TSP models for each elf group. We provide an initial solution (the greedy one created above) to initialize the model. In my tests, this helped the performance of the model. The TSP uses a Miller Tucker Zemlin formulation to eliminate subtours. You can change the time limit and gap limit directly in this procedure.
    * I applied a domain cut to the transport variable in the model. This was done setting parameters P_MaxDistance and P_EXE_MaxDistanceFreeAndForced. Both impact the P_DOM_V_Transport parameter which limits the variable that connects 2 nodes. This was also done to run faster and can eliminate good solutions as well.
4. I did not explore wildcards which could potentially improve the solution.

## Initial results

In a test run with Gurobi 9.5 for 20 minutes for each elf group I was able to obtain 2704 (4.75% gap / 2589 lower bound), 2718 (8,16% gap / 2541 lower bound) and 2558 (4.84% gap / 2434 lower bound) for each individual group. This means my solution was 2718. Current best solutions are 2428. If you compare this with lower bounds of the MIP models, the heuristic approach does cut out good solutions, so there is plenty of room to improve the algorithm.

I did not test CBC or CPLEX. If you are struggling with your solver, please feel to reach out in the AIMMS community (see link below).

 ## Possible improvements
 
1. Implement a heuristic or model that uses the wildcard to improve the solution.
2. Create a new processing step in the run heuristic to join all elf groups in a single model run so the solver can find improvements.
3. Improve the initial greedy heuristic. A suggestion is to look at the Kaggle foruns where there are many suggestions on algorithms for this.
 
 ## References
https://community.aimms.com/aimms-developer-12/santa-2021-the-merry-movie-montage-in-aimms-1124 - The AIMMS community discussing this challenge 

https://www.kaggle.com/c/santa-2021/overview - The kaggle challenge

https://how-to.aimms.com/Articles/332/332-Miller-Tucker-Zemlin-formulation.html#miller-tucker-zemlin-formulation - A description of the Miller Tucker Zemlin forumalation for TSP
 
