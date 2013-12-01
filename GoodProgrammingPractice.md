
# Good Programming Practice

We will start with a script that simulates predator-prey behavior via
an implementation of the Lotka-Volterra equations using NumPy, SciPy
and matplotlib.   

We'll clean up that script into a program, exploring aspects of good
programming practice along the way.

## Conform to style guidelines

First, we will pull our `import` statements up to the top of our file.

    import numpy as np
    from matplotlib import pyplot as plt
    from scipy import integrate

This conforms to the advice in [PEP 8 -- Style Guide for Python
Code](http://www.python.org/dev/peps/pep-0008/). Many languages have
style guides which specify conventions as to how constants, variables
and functions should be named, how code should be indented, what types
of code goes where etc. These are designed to help us produce readable
code that is understandable by others (and ourselves 6 months from
now). By conforming to a style guide, we ensure our code is consistent
in its presentation with that written by others. Once someone has
learned the style for a particular language,they can more readily
recognise what parts of the code does what, regardless of who wrote
the code - providing they conformed to the style guide.

While code is ultimately destined to be processed by a computer, it is
humans that spend a significant amount of time reading and writing it,
and your time is more valuable than computer time, so anything that
makes the job of reading and writing code easier is a good thing!

Now, we run our script to see that it still works after our changes.

## Modularise into functions

Our program consists of two main parts, that which does the
computation, and that which plots the results. We can pull out the
code that does the computation into a new function:

    def simulate():
        t = np.linspace(0, 20, 2000)
        X_initial = np.array([20, 4])
        X = integrate.odeint(dX_dt, X_initial, t)
        prey = X[:, 0]
        predators = X[:, 1]
        return (t, prey, predators)

As it's a function, we now need to return the time series and the
predator and prey populations at each time-step. We return these as a
3 element tuple. 

Likewise, we can pull out the code that plots the results into a new
function.  

    def plot(t, prey, predators):
        fig = plt.figure()
        plt.plot(t, prey, 'r-', label='Prey')
        plt.plot(t, predators, 'b-', label='Predators')
        plt.grid()
        plt.legend(loc='best')
        plt.xlabel('time')
        plt.ylabel('population')
        plt.title('Evolution of predator and prey populations')
        plt.show()

Finally, we need to add a bit of 'glue':

    (t, prey, predators) =  simulate()
    plot(t, prey, predators)

We now run this to see that it still works.

Why do this?  Well, we've decoupled how our computation is done from
how the results are presented. We are now free to change one or the
other without having to change both, so long as the *interface* each
function presents to the rest of the program does not change.

## Remove dependence on global variables

We have a number of variables that are global, they are used within
our functions but are assumed to have been defined elsewhere. Global
variables can be a problem:

 * They can be modified from anywhere within the program which can
make it difficult to understand how a program works.
 * They create mutual dependencies across a program - any part of a
program can change a global variable and any part of a program might
use a global variable.
 * They make it more difficult to reuse parts of a program in other
contexts e.g. as useful library such as we'll create shortly.

We'll start with `dX_dt`. Let's add arguments to the function, so we
know, when looking at the function body, where those values come from:

    def dX_dt(X, t, a, b, c, d):
        return np.array([ a*X[0] - b*X[0]*X[1] , -c*X[1] + d*b*X[0]*X[1] ])

Now we need to change the call to this function:

    X = integrate.odeint(dX_dt, X_initial, t, args=(a, b, c, d))

We use this `args` notation here as a consequence of how `odeint` is
defined. `odeint` unpacks `args` and passes them to `dX_dt` as a
conventional function call.

We now run this to see that it still works.

## Remove hard-coded values

We have hard-coded values for the initial populations of prey and
predators and the number of time-steps and the end-time.  If we wanted
to change the number of time steps or end time we'd have to edit our
function, so let's pass those in as arguments too.

Update `simulate` and pass in the initial populations of predators and
prey, the end time and the number of time steps. Remember to update
the glue code too!

Rather than run a simulation over a specific number of steps to a
specific end-time we can now choose how many steps and till when it
runs. We've made our function more general, flexible, and useful.

## Modularise into modules

We now have a single script that consists of functions and code that
is executed directly. We may want to use these functions in other
scripts, so we can pull them out into a new Python module.

Create a new file, `lotkavolterra.py`, and copy the `import`
statements and the functions into it.

Now remove the functions and redundant `import` statements from your
script and run it again.

We get an error as our functions are now defined outside our
script. We need to import them:

    from lotkavolterra import simulate
    from lotkavolterra import plot

If we run it again it should still work. 

We could now give this module to a colleague and they can use our
functions within their own programs should they wish. It's also
separated the computation and plotting code from the code that sets up
the initial values of the simulation, so we can them implement various
ways of setting up these initial values (e.g. by having one script
read them from the command-line or another read them from a
configuration file).

## Rename variables to be more meaningful

Our variable and function names in `lotkavolterra.py` are somewhat
cryptic. Variable names that are too short can be too cryptic, but if
they're too long this can inhibit comprehension. 12 characters or less
is recommended. Readable variable names also help our code to be
self-documenting.

Let's rename:  

* `dX_dt` - update predator and prey populations
* `X` - predator and prey populations
* `t` (in `dX_dt`) - time step
* `t` (in `simulate`) - time series

We'll deal with the rest shortly.

## Identify where data structures can be introduced

In the same way in which we bundled related code into functions, we
can bundle related variables into data structures.

If we look at the argument list to `simulate` is getting rather big -
it has 8 arguments - which can impact upon readability, since we are
programmed typically to hold only 7+/-2 chunks of information in
short-term memory at any time.

We can reduce the number of arguments by recognising that the
arguments group into those for prey and those for prey, plus the end
time and time-series. 

Let's use a Python dictionary to store the configuration values for
prey and predators. A Python dictionary is a set of key-value
pairs. If we give the dictionary a key, we get back the associated
value. Dictionaries are similar to associative arrays or hashtables in
other languages.

Let us first define some keys in `lotkavolterra.py`:

    BIRTH = "birth"
    DEATH = "death"
    POPULATION = "population"

In your script, import these keys:

    from lotkavolterra import BIRTH
    from lotkavolterra import DEATH  
    from lotkavolterra import POPULATION

Then create two dictionaries:

    prey_config = {}
    predator_config = {}

Now, update your script to populate the dictionaries e.g.

    prey_config[BIRTH] = a

Remember:

* `a` is the prey birth rate.
* `b` is the prey death rate.
* `c` is the predator death rate.
* `d` is the predator birth rate.

Update the functions to replace the 6 arguments relating to prey and
predators with two arguments, `prey_config` and `predator_config` and
to pull out values from the dictionaries when needed:

    prey_config[BIRTH]

If you get stuck as someone sitting by you - a fresh pair of eyes on
code can work wonders. Fagan (1976) discovered that a rigorous
inspection can remove 60-90% of errors before the first test is
run. M.E., Fagan (1976). [Design and Code inspections to reduce errors
in program development](http://www.mfagan.com/pdfs/ibmfagan.pdf). IBM
Systems Journal 15 (3): pp. 182-211. 

We currently return a tuple from `simulate`. Our time-series together
with the prey and predator populations together constitute the data
from our simulation. We can combine these into a single NumPy array
that contains one row per time-step where each row has the time, prey
population and predator population at that time-step.

The array returned by `integrate.odeint` already contains the prey and
predator populations at each time-step, and we have each time-step in
the array returned by `np.linspace` so we can insert the time-steps as
a first column by doing: 

    data = np.insert(populations, 0, time_series, axis=1)

This inserts `time_series` into `populations` at index 0 of the 1st
axis (as our data is 2D it inserts `time_series` as a new first
column). As this may be a bit cryptic to someone not familiar with
NumPy we can add comment before this line:

    # Insert time_series as a new first column.

Comments in-code should be used sparingly, to tell us why the code as
it is and also to make something which may be confusing, clear. 

Now just return this instead of a tuple and remove the, now redundant,
lines:  

    prey = X[:, 0]
    predators = X[:, 1]

Now change `plot` to take in a single argument and then unpack the
data. To unpack the data, use: 

* Time series - `data[:,0]`
* Prey values - `data[:,1]`
* Predator values - `data[:,2]`

Finally, change your script to reflect the fact that `simulate`
returns a single array and `plot` takes in a single array. 

## Save the simulation data

Our program runs the Lotka-Volterra equations to simulate the
evoluation of prey and predators over time. We could save our image if
we wish but then we would not be able to perform further analyses upon
it (and nor would our colleagues). So what we can do is save the raw
data itself. NumPy again helps us here (remember to add an `import`): 

    np.savetxt("data.csv", data, delimiter=",")

If we look at our data file:

    head data.csv

It has no context, nothing. What does this data mean? We know, as
we've just created it, but would we remember 6 months from now? We can
add information to our file to record:

 * What the data is.
 * What produced it i.e. the provenance of this data.

`savetxt` allows us to add a commented header to the data file. We can
use this, plus Python functions to add both the name of our program
plus the current date and time to this header:

    import os

    header = "Predator-prey Lotka-Volterra simulation data\n"
    header += "Produced by " + os.path.basename(__file__) + " on " + \
        time.asctime( time.localtime(time.time()))
    header += "\nTime-step, predator population, prey population"
    np.savetxt(output_file, results, delimiter=",", header=header)

Now, rerun the script and look at the data file. We now know when we
ran this script, what script was run, and what the data is. We can
load this into Python using `np.loadtxt` which will ignore the lines
beginning with `#` which are treated as comments.

As a quick example, using IPython:

    ipython --pylab
    data = np.loadtxt('data.csv', delimiter=',')
    plot(data[:,0],data[:,1])
    plot(data[:,0],data[:,2])

## More enhancements

The following files contain further enhancements to the code:

 * Library of simulation and plotting functions - [lotkavolterra.py](lotkavolterra/lotkavolterra.py)
 * Script to read configuration file, run simulation and save data file - [simulate_lv.py](lotkavolterra/simulate_lv.py)
 * Configuration file for the above - [config.cfg](lotkavolterra/config.cfg)
 * Script to read data file and plot data - [plot_lv.py](lotkavolterra/plot_lv.py)

## Key points

 * Write code for humans not computers.
 * Conformance to programming style guides helps others to understand
your code. 
 * Sensible variable and function names help make code
self-documenting. 
 * Breaking up code into functions, modules and packages promotes the
development of code that is easier to understand, modify and fix, and
reuse. 
 * Likewise, creating data-types out of conceptually-related groups of variables.
 * Add comments to explain why the code as it is or to explain code
that might otherwise be cryptic 
 * Record meta-data and provenance information in data files so you
have an audit trail.
