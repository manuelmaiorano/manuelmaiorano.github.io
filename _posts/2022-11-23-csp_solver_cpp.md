---
title: "CSP Solver with C++"
date: 2022-11-23
categories: C++ 
---

This project was inspired by the chapter on Constraint Satisfaction Problems of the book AI: A modern Approach. A CSP can be described in terms of variables that can assume values in a certain domain while satisfying a set of constraints. A constraint between two variables can reduce the domain of the variables. A problem is said infeasible if no assignment of values to variables can satisfy the constraints. The approch to solve this kind of prolems is based on searching the space of possible assignments. A way to speed up the search by cutting down (pruning) assignment that cannot possibly satisfy the problem, is to do forward inference. The way it works is by recursively shrinking the domain of the affected variables after a possible assignment; if at any point a variable's domain has size 0 the that assignment doesn't solve the problem and that branch can be discarded. The algorithm for recursively updating the domains of the variables is based on the famous AC3 algorithm, that works on a graph-like rapresentation of the constraints between variables. 

This project helped me improve my C++ skills, handling memory management, working with data structures and implementing from scratch an algorithm. Furthermore i learnt how to do configuration management in C++ using CMake, and how to create a static library structuring my CMakeLists.txt file in a hierarchical way, so that the code using the library known automatically which header files to include. The following is an excerpt of the main backtracking algorithm:

```cpp
bool MAC(Problem& csp, map<string, set<string>>& domain, string& assignedVar){

    queue<pair<string, string>>  starting_queue;
    for(auto& neigh: csp.GetNeigh(assignedVar)){
        if(domain[neigh].size() > 1)
            starting_queue.push({neigh, assignedVar});
    }
    
    while(!starting_queue.empty()){
        pair<string, string> arc = starting_queue.front();
        starting_queue.pop();
        string& Xi = arc.first;
        string& Xj = arc.second;
        if(ReviseDomain(csp, arc, domain[Xi], domain[Xj])){
            if(domain[Xi].empty()) return false;
            for(const string& Xk: csp.GetNeigh(Xi)){
                if(Xk != Xj)  starting_queue.push({Xk, Xi});
            }
        }
    }

    return true;

}

string selectUnassigned(map<string, set<string>>& currDomain, int domainSize, map<string, string> assignment){
    int min_size = domainSize;
    map<string, set<string>>::iterator it = currDomain.begin();
    string currVar = it->first;
    for(auto& [var,dom]: currDomain){
        int vecsize = dom.size();
        if(vecsize <= min_size && assignment.count(var) == 0){
            min_size = vecsize;
            currVar = var;
        }
    }
    return currVar;
}

bool Solver::BackTrack(Problem& csp, map<string, set<string>>& currDomain, function<void(int)> callback){
    if(assignment.size() == csp.nVariables()) return true;
    if (callback){
        callback(assignment.size());
    }
    string var = selectUnassigned(currDomain, csp.domainSize(), assignment);
    for(auto& value: currDomain[var]){
        map<string, set<string>> domainCopy = currDomain;
        domainCopy[var] = {value};
        assignment[var] = value;
        bool isValid = MAC(csp, domainCopy, var);
        if(!isValid) {
            assignment.erase(var);
            continue;
        }
        if(BackTrack(csp, domainCopy, callback))  return true;
    }
    assignment.erase(var);
    return false;
}
```
Here's an example run on two popular CSP problems (the map coloring problem and Sudoku):
```bash
cd examples
.\MapColoringProblem.exe
NSW : red
NT : red
Q : blue
SA : green
T : blue
V : blue
WA : blue
```
a sudoku:
```bash
.\SudokuProblem.exe
0 0 0  | 0 0 7  | 0 9 0 
0 0 0  | 0 2 0  | 0 0 8
0 0 9  | 6 0 0  | 5 0 0
-------+--------+------
0 0 5  | 3 0 0  | 9 0 0
0 1 0  | 0 8 0  | 0 0 2 
6 0 0  | 0 0 4  | 0 0 0
-------+--------+------
3 0 0  | 0 0 0  | 0 0 0 
0 4 0  | 0 0 0  | 0 0 0
0 0 7  | 0 0 0  | 3 0 0

percentage: 100%
8 3 6  | 5 4 7  | 2 9 1
7 5 4  | 1 2 9  | 6 3 8
1 2 9  | 6 3 8  | 5 4 7 
-------+--------+------
4 7 5  | 3 1 2  | 9 8 6
9 1 3  | 7 8 6  | 4 5 2 
6 8 2  | 9 5 4  | 1 7 3
-------+--------+------
3 6 8  | 4 9 1  | 7 2 5
5 4 1  | 2 7 3  | 8 6 9 
2 9 7  | 8 6 5  | 3 1 4
```
