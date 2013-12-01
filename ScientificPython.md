
# Scientific programming in Python

In this session, we will provide an introduction to Python and its support for scientific programming.

We will use [IPython](http://ipython.org/), an interactive computing shell built on top of Python. Two useful features are auto-completion and command history, analogous to what is provided by the shell. We will explore more features in due course.

Let us start ipython:

    ipython --pylab

We add the ``--pylab`` flag as this configures IPython with the libraries needed for interactive plotting.

## Implementing a dot product function

Scientists write a lot of software to simulate physical phenomena like diffusion-limited aggregation, but a lot more to crunch numbers. From the matrices used to calculate the stress and strain on a bridge to the tables used in statistics, the bulk of scientific data lives in arrays of one kind or another.

We can define an array of data, using a Python list:

    vector = [2,4,6]
    len(vector)

We can iterate over the list using a loop:

    for i in vector: print i

In Python a semi-colon `:` denotes a new block.

Python is not typed:

    vector = "hello world"

Like the bash shell, we can use up-arrow in IPython to get to the previous commands executed:

    for i in vector: print i
    vector = "123"
    for i in vector: print i

We get an error as an `int` is not iterable. The lack of explicit types can give rise to run-time errors like this, errors that can be caught at compile-time in C or Java.

Let us define two vectors:

    left = [2,4,6]
    right = [1,3,5]

Let us calculate the dot product of the vectors. For this we will use the `range` function:

    print range(len(left))

It gives us a list of consecutive values, from 0 up to, but not including, the value of its argument, in this case 3. 

We can use these as indices into the vectors. 

Our dot product is calculated by:

    product = 0.0
    for i in range(len(left)):
        product += left[i] * right[i]
    print product

In Python blocks are indented. Research into program comprehension has shown that people use white-space, not brackets, to determine what blocks of code belong to what conditions, loops, and functions. The Python designers force us to indent, so force us to write readable, or less unreadable, code.

Let us define a function to calculate the dot product, to save retyping this code:

    def dot(left, right):
        '''Calculate dot product of two equal-sized vectors.'''
        assert len(left) == len(right), 'Vector lengths unequal: {0} vs. {1}'.format(len(left), len(right))
        result = 0.0
        for i in range(len(left)):
            result += left[i] * right[i]
        return result

Now we can call the function:

    dot(left, right)

Let's cross-check against the result already calculated:

    print product

''' denotes a comment. Python can display this at the prompt:

    print dot.__doc__

`assert` checks that a pre-condition holds, the two vectors should be of equal length, and if not throws an error:

    dot([2,4,6], [1, 3])

Here the error causes our program to stop. In more complex programs, we can *catch* the error and *handle* it e.g. by popping up a dialog box to the user if we had a graphical user interface.

Writing code like this is inefficient in two senses. First, the same handful of vector and matrix operations come up so often that it's worth developing a special notation for them. Second, because those operations are common, it's worth investing time in optimizing their performance. Thousands of software developers have done exactly that over fifty years, producing libraries that are much faster, and much more reliable, than anything a single person could develop. These libraries are typically written in low-level languages like Fortran and C, and then wrapped up in MATLAB or Python to make them easier to use. For Python, one such library is NumPy.

## Efficient data processing and NumPy

[NumPy](http://www.numpy.org/) terms itself the "fundamental package for scientific computing with Python". NumPy  includes linear algebra, Fourier transform and random number capabilities. 

When we run IPython with `--pylab` the NumPy package, `numpy`, is already loaded and available under the alias `np`.

NumPy has a built-in dot product function. Let us look at its documentation:

    print np.dot.__doc__

Let us apply it to our Python lists:

    np.dot(left, right)

A Python list, [0, 1, 2], is actually an array of pointers to Python objects representing each number. At NumPy's heart is an optimised N-dimensional array object. NumPy arrays differ from Python lists, and tuples, in that the data is contiguous in memory. This allows NumPy arrays to be considerably faster for numerical operations than Python lists or tuples.

We can convert Python lists to NumPy arrays as follows:

    larray = np.array(left)
    rarray = np.array(right)
    type(left)
    type(larray)

`larray` and `rarray` are not lists but N-dimensional array, or ```ndarray```s.

Let's calculate the dot product. `dot` is *polymorphic* so can take either Python lists or NumPy arrays as arguments:

    np.dot(larray, rarray)

Let's create a new array and inspect it:

    vector = np.array([1,4,9,16])
    type(vector) # Type of vector
    vector.dtype # Type of vector's members - all members must be the same type
    vector.shape # Shape of vector - size along each axis, in this case 4 as it has 4 elements along the first axis

`#` is a Python comment.

We can create an array with members of different types. NumPy will convert everything to the most general type:

    vector = np.array([1.0, 2, 3])
    print vector
    vector.dtype

Note how everything is now a float. 

NumPy does not display the `0` in `1.0` as a way of keeping the output concise.

We can force everything to be a particular data type by providing an optional argument:

    vector = np.array([1, 2, 3], dtype=np.float32)
    print vector
    vector.dtype

### Creating arrays in-code

We won't normally type in all our data. Instead, we will either construct arrays in stereotyped ways, or read data from files. Here are some examples of the first approach as NumPy comes with a number of useful functions to create arrays of a given size and initial values.

    z = np.zeros((5, 3))
    print z
    print np.zeros((5, 3, 2))
    print np.ones((5, 3))
    print np.identity(5)

In NumPy, array dimensions are expressed as a Python *tuple* - unlike in a list, or array, a tuple's items cannot be re-assigned. Here we are giving `zeros` and `ones` exactly 1 argument, a tuple.

Note that the shape (5,3) does not mean "5 elements along the X axis, and 3 along the Y". Instead, it means "5 along the first axis, and 3 along the second". This is called row-major order, and when we print the array, NumPy shows us 5 sub-arrays, each containing 3 elements.

We can create multi-dimensional arrays using lists of lists:

    rectangle = np.array([[11, 22], [33, 44], [55, 66]])
    print rectangle

### Aliasing

Let's try to copy an array and then change a value in the copy:

    first = np.ones((2, 2))
    print first
    second = first
    print second
    second[0, 0] = 9
    print second
    print first

Why has `first` changed? It has changed because the data in `first` was not copied to `second`. Rather, `second` was changed to point at the same place in memory as `first`. This is similar to pointers in C or object references in Java. In NumPy this is called *aliasing*. Aliasing is done as:

 * It is more efficient that copying data unnecessarily, and
 * that's what Python does in other cases e.g. with lists.

If we want to copy data so that we can safely make changes, we can do that explicitly using the array object's `copy` method.

    third = first.copy()
    print third
    third[1, 1] = 1234
    print third
    print first

### Performance

We said that NumPy arrays give us performance. Let us see if that is the case and create a Python list and a NumPy array, each with a million occurences of the number `1`. First let's create a list of 1000000 elements, using Python short-hand:

    million = 1000000
    mill_list = [1] * million
    len(mill_list)

Now, let's create a NumPy array of 1000000 1s:

    mill_parray = np.ones((million,))
    len(mill_array)

We can now use IPython's `timeit` command to see how long it takes to sum up the values in the list:

    %timeit sum(mill_list)

And in the array:

    %timeit np.sum(mill_parray)

Note that `timeit` is an IPython [magic function](http://ipython.org/ipython-doc/dev/interactive/tutorial.html) not a Python command. This is denoted by the `%` - IPython has *cell magics*, denoted by `%`, which take the rest of the line as an argument, and *line magics*, denoted by `%%` which take the following lines, until a blank line, as arguments. So we can also run the above as:

    %%timeit
    np.sum(mill_array)

Magic functions have built-in help, for example:

    %timeit?

### Analyzing patient data

The other way to create arrays is to read data from files. NumPy has functions to handle a wide variety of common formats, including comma-separated values (CSV). Suppose we have a file called patients.csv that contains normalized white blood cell counts for 60 people during the 40 days after contact with someone carrying drug-resistant tuberculosis (DRTB). We can use shell commands to look at how large the file is, and the first few lines in it.

Rather than having to kick up a new terminal window, IPython allows us to run shell commands using `!`:

    !wc -l patients/patients.csv
    !head -5 patients/patients.csv

We will load in this data file using NumPy's `loadtxt` function. We provide an optional delimiter argument to tell it that the values are separated by commas:

    patients = np.loadtxt('patients/patients.csv', delimiter=',')
    patients.shape
    print patients

Note the ellipsis, `...`, that NumPy uses when displaying a very large array.

Let us look at the values for a single patient:

    p0 = patients[0, :]
    len(p0)
    print p0

`[0, :]` is *slice syntax*. Here we are saying that we want the first, 0th, set of data from the first axis, then all the data from the second axis. Or, as our data is 2-dimensional, want the entire first row only.

Let us look at the white cell counts at time t=0 for all patients:

    t0 = patients[:, 0]
    len(t0)
    print t0

`[:, 0]` says that we want all the data from the first axis, then, from that the first, 0th, set of data from the second axis. Or, as our data is 2-dimensional, we want the entire first column only.

We can calculate the average white cell count for all patients across the entire 40 days:

    np.mean(patients)

A more meaningful statistic is probably the mean white cell count over time across all our patients. To get this, we  tell NumPy which axis we want it to sweep over. In this case, that is the first axis, 0, because that's the one that distinguishes patients from each other:

    mean_over_time = np.mean(patients, 0)
    print mean_over_time

We can check that we've done the calculation along the right axis by looking at the size of our result which should match the number of days, 40:

    len(mean_over_time)

Simlarly, we can calculate the average white cell count for each patient over all time by asking NumPy to calculate the mean over the second axis:

    patient_means = np.mean(patients, 1)
    print patient_means
    len(patient_means)

In many cases, we will now want to calculate statistics on all of our values, but only on those that meet certain criteria. For example, let us see which patients had a normalized white cell count of 0 on the first day of exposure:

    print patients[:, 0] == 0

We can do a visual inspection but suppose we had 100 or 1000 patients. We should always automate such monotonous, potentially error-prone activity and let the computer do the work. Here, we can check using `np.all`, which tells us whether all the elements in an array are true:

    np.all(patients[:, 0] == 0)

Let's try that again for day 1:

    np.all(patients[:, 1] == 0)
    np.sum(patients[:, 1] == 0)

That last line gives us the number of uninfected patients. NumPy treats `True` as `1` and `False` as `0`:

    print True == 1
    print False == 0

We can use these conditions to select data from our array, for example the number of infected patients on day 0:

    sample = patients[ patients[:, 0] > 0 ]
    sample.shape

And on day 1:

    sample = patients[ patients[:, 1] > 0 ]
    sample.shape

NumPy uses the array of 40 `True` and `False` as a mask., lines them up with the major axis of `patients`, and gives us just those rows where the mask is `True`. We can now do more arithmetic with this sub-array, such as finding the maximum white cell count for those people who were showing signs of infection on day 1:

    max = np.max(sample, 1)
    print max

and then calculate the average maximum cell count for those people:

    print np.average(max)

and compare it with the average maximum cell count for people who weren't showing signs of infection on day 1:

    print np.average(np.max(patients[ patients[:, 1] == 0 ], 1))

### Readability versus efficiency

Our code highlights the simultaneous strength and weakness of using array operators. In `patients/patient-average.py` there is an example of code which calculates our average maximum cell count for the uninfected on day 0:

    total = 0.0
    num = 0
    for p in range(60):
        if patients[p, 1] == 0:
            max_count = 0
            for t in range(40):
                if patients[p, t] > max_count:
                    max_count = patients[p, t]
            total += max_count
            num += 1
    print 'result', total / num # We expect 17.5757575758

We can look at this using:

    !cat patients/patient-average.py

We can run it using:

    %run -i patients/patient-average.py

`-i` runs in interactive mode meaning that the script can assume we have already set the value of variables such as `patients` when running interactively.

Compare this to our expression:

    np.average(np.max(patients[ patients[:, 1] == 0], 1))

which is efficient, but does take a bit of practice to read, and it's very easy to fail to notice the difference between == and !=, or axis 0 versus axis 1, when they're buried inside a complex expression. 

We'll look more at readability in our session on Good programming practice.

## Visualization and matplotlib

The mathematician Richard Hamming once said, "The purpose of computing is insight, not numbers," and the best way to develop insight is often to visualize data. Visualization deserves an entire lecture (or course) of its own, but we can explore a few features of Python's 2D plotting library, [matplotlib](http://matplotlib.org), here. 

When we run IPython with `--pylab` the Matplotlib package, `pyplot`, is already loaded and available under the alias `plt`.

Now let us plot our patient data:

    plt.imshow(patients)
    plt.show()

This plot shows how the white cell counts rise and fall for our patients over the 40 day period. They seem to rise and fall quite predictably. 

Let's close that figure.

    plt.close()

Let's now take a look at the average degree of infection over time:

    n_patients, n_days = patients.shape
    dates = range(n_days)
    print dates
    avg_infection = np.average(patients, 0)
    plt.figure()
    plt.plot(dates, avg_infection)
    plt.show()

The graph above is surprisingly regular. Let's try looking at the maximum cell count per day across all our patients:

    plt.plot(dates, np.max(patients, 0))
    plt.show()

Whoops: that's more than surprising, it's downright suspicious. What does the minimum look like?

    plt.plot(dates, np.min(patients, 0))
    plt.show()

Again, that is suspiciously regular. This is because our patient data was produced by a random number generator producing uniform values between D/4 and D, where D ramps up and down uniformly from the start to the end of our 40-day period. If we were reviewing a paper that used this data, now would be the time to notify the editor of our suspicions.

If we're going to do that, though, we probably ought to tidy up our plots. First, we'll put them side by side:

    plt.close()
    plt.figure()
    plt.subplot(1, 3, 1)
    plt.subplot(dates, avg_infection)
    plt.show()
    plt.subplot(1, 3, 2)
    plt.plot(dates, np.max(patients, 0))
    plt.show()
    plt.subplot(1, 3, 3)
    plt.plot(dates, np.min(patients, 0))
    plt.show()

`subplot` creates a new sub-plot: all subsequent plotting commands apply to it until a new sub-plot is created. The three arguments tell the library how many rows and columns of sub-plots we want, and which sub-plot this is. This figure has all our information, but it looks a bit squashed, and it would be helpful to have some titles. Let's make some formatting changes:

    plt.subplot(1, 3, 1)
    plt.xlabel('date')
    plt.ylabel('average')
    plt.subplot(1, 3, 2)
    plt.xlabel('date')
    plt.ylabel('maximum')
    plt.subplot(1, 3, 3)
    plt.xlabel('date')
    plt.ylabel('minimum')
    plt.tight_layout()

`xlabel` and `ylabel` put labels on the axes, and `tight_layout` makes sure that there's enough space between the figures that the vertical labels don't overlap the adjacent figures.

    plt.close()

## Scientific programming and SciPy

[SciPy](http://scipy.org) is an open source library of Python tools for mathematics, science and engineering. We've already been using three of these tools: UIPython, NumPy and matplotlib. 

We can see what ScipPy offers by viewing the packages it provides:

    import scipy
    print scipy.__doc__

Here, we'll use SciPy's ordinary differential equation (ODE) solver.

The [Lotka-Volterra](http://wiki.scipy.org/Cookbook/LoktaVolterraTutorial) equation, first proposed in the 1920s, models the interactions between predators and prey (e.g. cats and mice, foxes and rabbits). When predators are scarce, prey breed rapidly; as more prey become available, the predator population increases; and as the number of predators increases, prey become scarcer, so the predator population peaks and falls. In mathematical form, this is:

    du/dt =  a*u -   b*u*v

    dv/dt = -c*v + d*b*u*v

(or see it in [mathematical notation](lotkavolterra/lotkavolterra.png))

where:

* u is the number of prey.
* v is the number of predators.
* a is the natural growth rate of prey when there are no predators.
* b is the natural death rate of prey due to predation.
* c is the natural death rate of predators when there are no prey.
* d describes how many prey have to be caught and eaten to produce a new predator.

Let's define this equation in Python. If X is the pair [u, v], the prey and predator populations, then the equation of change in population over time is:

    a = 1.0
    b = 0.1
    c = 1.5
    d = 0.75
    def dX_dt(X, t):
        return np.array([ a*X[0] - b*X[0]*X[1] , -c*X[1] + d*b*X[0]*X[1] ])

We create the state variable `X` in order to use the numerical integration function we'll introduce in a moment. Similarly, the function that calculates the derivative, `dX_dt`, has to take the state and the time as parameters, even though it doesn't use the latter.

Let us integrate our equation from t = 0 to t = 20 in 2000 time-steps. To do this, we need a vector of time values, which we can create using NumPy's `linspace` function, which returns a list of evenly spaced numbers over an interval:

    t = np.linspace(0, 20, 2000)

Let's start with a population of 20 rabbits and 4 foxes:

    X_initial = np.array([20, 4])

We can now integrate numerically using `odeint`, the ordinary differential equation integrator, from the SciPy library. First we need to import it. 

    from scipy import integrate

`import` imports functions, variables or collections of these from elsewhere. This is similar to the `import` command in Java or `include` in C. 

Now we can run it:

    X = integrate.odeint(dX_dt, X_initial, t)

`X` is now 2000 pairs of numbers representing the prey and predator populations at each time t:

    X.shape

Let us now plot these:

    prey = X[:, 0]
    predators = X[:, 1]
    plt.figure()
    plt.plot(t, prey, 'r-', label='Prey')
    plt.plot(t, predators, 'b-', label='Predators')
    plt.show()

Through trial and error we might have done some experimental data analysis and created a plot as we did above. How do we record all the commands we ran? IPython provides commands to allow us to do just that. Run:

    %history

Now, run the following to see the command history with line numbers:

    %history -n

We can save selected commands within a file using IPython's `%save` command. So save your Lotka-Volterra commands, specifying the line number where you ran:

    a = 1.0

and the line number where you ran:

    plt.show()

For example:

    %save simplelotka.py 26-48

Now use IPython to invoke the shell's `cat` command to see that the file is there:

    !cat simplelotka.py

Once it is, enter:

    exit()

to exit IPython.

Now we can run it:

    python simplelotka.py

It'll fail. Use the error message to figure out why (clue, you need to `import` some things) and clean up your script so that when you run:

    python simplelotka.py

it displays your plot.

Add in commands to display labels on the X and Y axes, a title and a legend (e.g. "Evolution of predator and prey populations")

Now, we have a script that does our analysis that we can rerun as and when desired.

## Key points

 * If you're using a matrix library and end up writing a loop, chances are you're doing something wrong.
 * A number of powerful scientific libraries are available for Python.
 * Not-invented-here can incur a lot of reinventing the wheel, more poorly than existing inventions.
 * A lot of software development involves taking off-the-shelf libraries and gluing them together.
 * Readability versus efficiency is a commonly-occurring trade-off.
