> This assignment assumes you went through all lectures in the "OpenFOAM simulation in a nutshell" module

# Assignment for "OpenFOAM simulation in a nutshell" lectures

## Goals

- Become confident in basic OpenFOAM simulations
- Practice setting up OpenFOAM simulations
- Practice basic ParaView visualization mechanisms.

## Basic-level skills

If you haven't done so yet, clone the demo case and switch to `EndOfPreprocessing` stage:

```bash
> run
> git clone https://github.com/FOAM-School/res-eng-openfoam-intro intro
> cd intro
> git checkout EndOfPreprocessing
```

### Getting ready for the assignment

Before we start, I want you to make some changes to the case:

- Remove (Or comment out) the `maxIter  1;` from `fvSolution.solvers.T`
   dictionary first. The iterative solver then stops iterating.
- Set `tolerance  1e-7;` in the same dictionary.
- In `system/controlDict`, change `endTime` from 40 to 3.

Run `blockMesh && scalarTransportFoam` to get a grasp of the state of the simulation (forming a base line).

1. Note that, with the above setup, the iterative solver converges in the first "timestep". How many 
   iterations did it take? Hint: look for the line: `smoothSolver:  Solving for T ...` in solver output. 
2. Check **T** values in `2/T` for example. Better yet, run the following command to
   produce a table of **T** values similar to the ones shown in the lectures
   (Note that **T** doesn't change beyond the first iteration):
   ```bash
   > for Tdict in $(ls --sort version *[^g]/T); do grep -e "internalField" $Tdict; done
   ```

Now, clean the case to proceed: `foamCleanTutorials && blockMesh`

### Playing with "Initial Conditions" and simulation convergence

> Note that this is a steady-state simulation; By **Initial Conditions**,
> we mean the initial state of **T** variable for the iterative solver (Gauss Seidel)

In the lectures, we used zeroed initial state for **T**. Let's change it to a more logical one:

```cpp
///   In 0/T
internalField nonuniform List<scalar> 9(0.95 0.9 0.8 0.7 0.6 0.5 0.4 0.2 0.1);
```
Run the case again and answer the folowing questions (based on solver log and T values in the first 3 timesteps):

1. How many iteration did it take to converge? How many "timesteps" were required?
2. Is there (Do you expect) any difference in the final solution (from the previous initial state)?
3. Fill in the following table and identify the stopping criterion (`tolerance` or `relTol`)

   | TimeStep | Initial Residual | Final Residual | Per-Timestep Convergence Criterion |
   |----------|------------------|----------------|--------------------|
   |    1st   |                  |                |                    |
   |    2nd   |                  |                |                    |
   |    3rd   |                  |                |                    |

Clean the case (`foamCleanTutorials && blockMesh`), and increase the velocity to (1 0 0)

4. Run the solver again and see what happens. Any guesses on why this happens? Hint: Watch the Courant number.

5. Run the solver again with the velocity set to (0.3 0 0). Can you deduce the maximum number of linear iterations allowed
   per timestep? (remember that we have *removed* `maxIter 1;` from `fvSolution.solvers.T`)
   
6. What we did in question 4 made the iterative method "incapable" of **solving** the system of equations.
   And question 5 made the iterative method "incapable" of converging. Which case is hardest to figure out?

7. Here is an even harder case, try U = (0.2 0 0). Does the simulation converge?
   What about the accuracy of the results? The theoretical solution has the same form as in the lecture but 
   `a = -3.04599e-7, b = 0.2`.

> I think I made the point here, convergence doesn't mean the result are accurate.
> Don't forget to get the velocity back value to (0.03 0 0). 

### Iterative solvers and preconditioners

Let's now experiment with more linear solvers. To figure out what kind of iterative solvers are available for the
**T** field, we can replace `fvSolution.solvers.T.solver` with "res-eng" (for example, we know there is no iterative 
solver named res-eng), and then run `scalarTransportFoam`.

This will cause OpenFOAM solvers to display a list of available iterative solvers for the field.
```bash
--> FOAM FATAL IO ERROR: 
Unknown asymmetric matrix solver res-eng
Valid asymmetric matrix solvers are :

12
(
BICCG
BiCG
BiCGStab
FPEAMG
GAMG
GMRES
MPEAMG
PBiCG
RREAMG
amgSolver
deflation
smoothSolver
)
```
> Different OpenFOAM forks will offer a different set of iterative solvers.

The error lists all available solvers for asymmetric matrices (which is the type of our matrix).

1. Instead of "res-eng" as the iterative solver, try **PBiCG** (short for Preconditioned Bi-conjugate gradient).
   Run `scalarTransportFoam` and see what happens (read through the error).

> Hint: PBiCG (and many others) require a "preconditioner" which is an operation applied to the matrix before 
> solving the system of equations to improve its condition number ( get
> <img src="https://latex.codecogs.com/gif.latex?cond(M)=|M^{-1}|.|M|" title="cond(M)=|M^{-1}|.|M|" />
> near the value 1, where **M** is the matrix in question)

2. You can now include `preconditioner    something;` in `fvSolution.solvers.T` dictionary to get a list 
   of available pre-conditioners.

3. Pick, `preconditioner   diagonal;` (The simplest one) and try the following iterative solvers 
   (All variants of the Krylov-space CG solver) and complete the table:

   | Iterative solver | Number of Iterations to converge | Final Residual |
   |------------------|----------------------------------|----------------|
   |       PBiCG      |                      | |
   |     BiCGStab     |                      | |

> If there is an interative solver you want to learn more about, the source code can give you
> a fairly sufficient amount of information (no C++ knowledge is required, just read the description)
> : `find $FOAM_SRC -iname "PBicG*H"` for example (the H is there to get only the "header files").

4. How these perform when compared to the Gauss Seidel Method?
5. Reservoir Engineers use more the GMRES (Generalized Minimal RESidual method, another Krylov-space method) 
   in their simulations.
   So, let's try it out. Set `fvSolution.solvers.T.solver` to GMRES, and run `scalarTransportFoam`.
   The iterative solver requires that you specify the Krylov-space dimensions 
   (`nDirections  5;` is what I usually do: we know 9 dimensions would solve the problem 
   in a single **iteration**. But, do try different values for `nDirections` - between 1 and 9 -).

6. Another family of iterative solvers (other than Krylov-Space ones) is the multi-grid family. Those
   are the iterative solvers that solve the matrix on different levels of mesh density (starting from 
   the most coarse mesh and building up to reach the finest one). An example of such solvers is **GAMG**
   (Geometric-Algebraic Multi-Grid) which can apply the agglomeration on mesh cells directly (the Geometric
   approach) or on matrix cofficients only (the Algebraic approach).
   Set `fvSolution.solvers.T.solver` to `GAMG` and `scalarTransportFoam`. You know the drill.
   Just any keywords that the solver says are missing, with a dummy value ("something" for example) to get
   a list of available options each time. It's recommended to pick the following configuration though:
   
   - Agglomeration type: `algebraicPair`
   - Number of cells in coarsest mesh level: sqrt(nCells) is usually good. In this case, sqrt(9) = 3.
     But do try 4 and 2 afterwards.
   - Merge agglomeration Levels should be set to 1 (enabled).
   - Keep the smoother to GaussSeidel.
   
> Note that we tried a whole bunch of iterative solvers without touching the smoother keyword.
> Some of these methods indeed required the presence of this keyword, and some of them didn't use it at all.
> But OpenFOAM didn't complain about it.
   
7. One last thing to do is to try out different preconditioners for this matrix. Set `fvSolution.solvers.T.solver`
   to **PBiCG** and change the preconditioner each time:
   
   | Preconditioner | Number of Iterations to converge | Final Residual |
   |------------------|----------------------------------|----------------|
   |      diagonal      |                      | |
   |      DILU     |                      | |
   |      SymGaussSeidel     |                      | |
  
## Intermediate-level skills

I think The basic-level section made it clear that the best configuration to use is the **PBiCG** as the solver and
**DILU** as its preconditioner. So, we'll continue with this configuration.

### Mesh Density

What if we increase cells number in our domain? from 9 to 20, 300, 10000. 

> Cell Size issues usually arise in transient simulations. As this is a steady-state one, we
> are relatively safe. We can go for 100000 cells without problems (capturing a change of the order 1e-5 in T value
> between cell centers).

The procedure to change the cell number is as follows:

- In `constant/blockMeshDict.blocks`, we change the 9 in `hex (0 1 2 3 4 5 6 7) (9 1 1) simpleGrading (1 1 1)`
  to whatever we like.
- Re-Build the mesh and check its quality (with `blockMesh && checkMesh`) 

### Using Jupyter Notebooks to manage case reports

## Advanced-level skills
