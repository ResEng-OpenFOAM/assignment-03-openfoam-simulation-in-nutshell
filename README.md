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
Run the case again and answer the folowing questions:

1. How many iteration did it take to converge? How many "timesteps" were required?
2. Is there (Do you expect) any difference in the final solution (from the previous initial state)?
3. Fill in the following table and identify the stopping criterion (`tolerance` or `relTol`)

   | TimeStep | Initial Residual | Final Residual | Per-Timestep Convergence Criterion |
   |----------|------------------|----------------|--------------------|
   |    1st   |                  |                |                    |
   |    2nd   |                  |                |                    |
   |    3rd   |                  |                |                    |



## Intermediate-level skills

Python tools
```bash
> export MPLLOCALFREETYPE=1
> apt install -y libpng-dev
> python3 -m pip install cython numpy matplotlib jupyter pandas
```

## Advanced-level skills
