# Notes on FOAM tests

## Compiling the code

To debug a solver in OpenFOAM, the code must be compiled with `WM_COMPILE_OPTION=Debug` set in `etc/bashrc`. This will set some debug options. For GNU's Gcc compiler, these debug options can be found and modified in `wmake/rules/linux64Gcc/c++Debug`.

## Debugging with GDB

To debug a solver type say `gdb icoFoam` or `gdb -tui icoFoam`. The latter brings up a UI embedded in the terminal.

## Debugging with GDB and VSCODE

Create the following files in a .vscode folder of the main wmake directory of the program and edit appropriately

**c_cpp_properties.json**

```json
{
  "configurations": [
    {
      "name": "Linux",
      "includePath": [
        "${workspaceFolder}/**",
        "${FOAM_SRC}",
        "${FOAM_SRC}/OpenFOAM/lnInclude",
        "${FOAM_SRC}/OSspecific/POSIX/lnInclude"
      ],
      "defines": [],
      "compilerPath": "/opt/OpenFOAM/ThirdParty-v1912/platforms/linux64/gcc-6.3.0/bin/gcc",
      "cStandard": "gnu11",
      "cppStandard": "gnu++14",
      "intelliSenseMode": "gcc-x64"
    }
  ],
  "version": 4
}
```

**launch.json**

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "name": "OF-Debug",
      "type": "cppdbg",
      "request": "launch",
      "program": "${env:FOAM_USER_APPBIN}/${fileBasenameNoExtension}",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${workspaceFolder}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "setupCommands": [
        {
          "description": "Enable pretty-printing for gdb",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "wmake-build",
      "miDebuggerPath": "/usr/bin/gdb"
    }
  ]
}
```

**tasks.json**

```json
{
  "tasks": [
    {
      "type": "shell",
      "label": "wmake-build",
      "command": "wmake",
      "options": {
        "cwd": "${workspaceFolder}"
      },
      "problemMatcher": ["$gcc"],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ],
  "version": "2.0.0"
}
```

To run gdb commands in VSCODE simply precede the command with -exec e.g.

```gdb
-exec info macro forAll
```

Typing `-exec help` will give

```
List of classes of commands:

aliases -- Aliases of other commands.
breakpoints -- Making program stop at certain points.
data -- Examining data.
files -- Specifying and examining files.
internals -- Maintenance commands.
obscure -- Obscure features.
running -- Running the program.
stack -- Examining the stack.
status -- Status inquiries.
support -- Support facilities.
tracepoints -- Tracing of program execution without stopping the program.
user-defined -- User-defined commands.

Type "help" followed by a class name for a list of commands in that class.
Type "help all" for the list of all commands.
Type "help" followed by command name for full documentation.
Type "apropos word" to search for commands related to "word".
Type "apropos -v word" for full documentation of commands related to "word".
Command name abbreviations are allowed if unambiguous.
```

Typing `-exec help all` will give a [full list of commands](GDBHelp.md)

A list of useful commands:

| Command             | Decription                                           | Example             | Result from Test-BinSum.C |
| ------------------- | ---------------------------------------------------- | ------------------- | ------------------------- |
| whatis              | obvious                                              | -exec whatis rndGen | type = Foam::Random       |
| backtrace full      | Complete backtrace with local variables              |                     |                           |
| up, down, frame     | Move through frames                                  |                     |                           |
| watch               | Suspend the process when a certain condition is met  |                     |                           |
| set print pretty on | Prints out prettily formatted C source code          |                     |                           |
| set logging on      | Log debugging session to show to others for support  |                     |                           |
| set print array on  | Pretty array printing                                |                     |                           |
| finish              | Continue till end of function                        |                     |                           |
| enable and disable  | Enable/disable breakpoints                           |                     |                           |
| tbreak              | Break once, and then remove the breakpoint           |                     |                           |
| where               | Line number currently being executed                 |                     |                           |
| info locals         | View all local variables                             |                     |                           |
| info args           | View all function arguments                          |                     |                           |
| list                | view source <lineno> (defaults to current) +-5 lines |                     |                           |
| rbreak              | break on function matching regular expression        |                     |                           |

<br>

## Example for Test-BinSum

1. Place a breakpoint manually in the vscode viewing at the first line and launch debugger. This breakpoint will highlight red in the vs-code viewer.
2. Switch on logging and tracing on by `-exec set logging on` `-exec set trace-commands on`. <br>By default this will copy output to `gdb.txt` and debug output to `gdb.txt`.
3. If using visual studio code, use the UI to specify breakpoints. If not you can use `b <lineno>`. Mixing these methods in vs-codes can lead to issues, so don't mix!
4. Place a breakpoint at line 48, run the debugger, and progress to line 51
   ```c++
   // This is printed by typing "-exec list 51" in vs-code)
   46	int main(int argc, char *argv[])
   47	{
   48	    Random rndGen(0);   // place breakpoint here
   49
   50	    scalarField samples(10000000);
   51	    forAll(samples, i)  // Continue until here
   52	    {
   53	        samples[i] = rndGen.sample01<scalar>();
   54	    }
   ```
5. Breakpoints can be reviewed with `-exec info breakpoints`. The viewer also provides info.

   ```verbatim
    Num     Type           Disp Enb Address            What
    2       breakpoint     keep y   0x000000000040439f in main(int, char**) at Test-BinSum.C:48
   breakpoint already hit 1 time
   ```

6. Let's take a closer look at rndGen, which is evidently of type `Random`. Typing `-exec ptype rndGen` returns

   ```c++
   type = class Foam::Random {
   private:
       Foam::label seed_;
       Foam::Rand48 generator_;
       std::uniform_real_distribution<double> uniform01_;
       bool hasGaussSample_;
       Foam::scalar gaussSample_;
   public:
       static const Foam::label defaultSeed;

   private:
       Foam::scalar scalar01(void);
   public:
       Random(Foam::label);
       Random(const Foam::Random &, bool);
       ~Random();
       Foam::label seed(void) const;
       void reset(Foam::label);
       int bit(void);
   }
   ```

   It can be seen that it has a private (?) member function called scalar01() which appears to be called on line 53. Let's try calling it with `-exec p rndGen.scalar01()` $\rightarrow$

   ```c++
   -exec p rndGen.scalar01()
   $4 = 0.78579925865740674
   -exec p rndGen.scalar01()
   $5 = 0.36876626996891299
   -exec p rndGen.scalar01()
   $6 = 0.74509509879811853
   ```

   If this is private, how can it be accessed. Looking at the `Random.H` we have the public member function which is facilitating the call.

   ```c++
   template<class Type>
   Type sample01();
   ```

   However calling the function with the templated format used on line 53 will not work.

   ```c++
   -exec p rndGen.scalar01<double>()
   Could not find method Foam::Random::scalar01<double>

   -exec p Foam::Random::sample01<double>
   {Foam::scalar (Foam::Random * const)} 0x7ffff78fecae <Foam::Random::sample01<double>()>

   ```

7. Looking at the member data of the rndGen generator, some data (e.g. `engine_`) is optimized out. This is because `engine_` is part of the `libstdc++` library which is unaffected by the `WM_COMPILE_OPTION=Debug` option.

   ```c++
   -exec p rndGen
   $8 = {seed_ = 0, generator_ = {engine_ = {static multiplier = <optimized out>, static increment = <optimized out>, static modulus = <optimized out>, static default_seed = <optimized out>, _M_x = 152005434852800}, static default_seed = 1}, uniform01_ = {_M_param = {_M_a = 0, _M_b = 1}}, hasGaussSample_ = false, gaussSample_ = 0, static defaultSeed = 123456}

   -exec ptype rndGen.generator_.engine_
   type = class std::linear_congruential_engine<unsigned long, 25214903917ul, 11ul, 281474976710656ul> [with _UIntType = unsigned long] {
     public:
       static _UIntType multiplier;
       static _UIntType increment;
       //...
   ```

8. So in summary lines 46:54 seem to be setting a scalarField `samples` with random values which are uniformly selected between 0 and 1.

9. Now navigate to line 63
   ```c++
   56	    const scalar min = 0;
   57	    const scalar max = 1;
   58	    const scalar delta = 0.1;
   59
   60	    BinSum<scalar, scalarField> count(min, max, delta);
   61	    BinSum<scalar, scalarField> sum(min, max, delta);
   62
   63	    forAll(samples, i)
   64	    {
   65	        count.add(samples[i], 1);
   66          sum.add(samples[i], samples[i]);
   67      }
   ```
10. At this point the scalarField `samples` should be filled with random numbers.

    ```c++
    -exec ptype samples
    type =
    class Foam::Field<double> [with Type = double] :
        public Foam::FieldBase, public Foam::List<Type>
    {
      public:
        // ...
    }

    -exec p samples
    $9 = {<Foam::FieldBase> = {<Foam::refCount> = {count_ = 0}, static typeName = 0x7ffff7d4e3de "Field", static allowConstructFromLargerSize = false}, <Foam::List<double>> = {<Foam::UList<double>> = {size_ = 10000000, v_ = 0x7ffff14cc010}, <No data fields>}, <No data fields>}
    ```

    Above we see that samples is of type `Field` which inherits from `List` which contains an array `v_` of size `size_`. Let's print some of the elements.

    ```c++
    -exec ptype samples.v_
    type = double * restrict

    -exec ptype samples.size_
    type = int

    -exec p samples.size_
    $14 = 10000000

    -exec p samples[0]
    $39 = (double &) @0x7ffff14cc010: 0.35372820284499129

    -exec p *samples.v_
    $15 = 0.35372820284499129

    -exec p *(samples.v_+1)
    $16 = 0.26022200134234486

    -exec p samples.v_[2]
    $17 = 0.77678995132180573

    -exec p *samples.v_@3
    $18 = {0.35372820284499129, 0.26022200134234486, 0.77678995132180573}

    -exec p *samples.v_@samples.size_
    value requires 80000000 bytes, which is more than max-value-size

    -exec set $printlen = 10
    -exec p *samples.v_@$printlen
    $19 = {0.35372820284499129, 0.26022200134234486, 0.77678995132180573, 0.57578820076074222, 0.083004246333399131, 0.10994681411228381, 0.39068657045496069, 0.95906644946364006, 0.16318951028758619, 0.26036109462171275}
    ```

    The above shows different ways to print the elements of the array v\_ in the `scalarField`. In the last two lines we create a variable to set the number of elements that we want to print to screen.

11. Now let's turn our attention to lines 55:64.

    ```c++
    // Using whatis to investigate
    -exec whatis min
    type = const Foam::scalar

    -exec whatis Foam::scalar
    type = const Foam::doubleScalar

    -exec whatis Foam::doubleScalar
    type = double

    // Using ptype to investigate
    -exec ptype min
    type = const double

    -exec ptype max
    type = const double

    -exec ptype count
    type =
    class Foam::BinSum<double, Foam::Field<double>,
                Foam::plusEqOp<double> > [with   IndexType = double, List = Foam::Field<double>, CombineOp = Foam::plusEqOp<double>] :
                public List
    {
      private:
        IndexType min_;
        IndexType max_;
        IndexType delta_;
        IndexType lowSum_;
        IndexType highSum_;

      public:
        BinSum(IndexType, IndexType, IndexType);
        BinSum(IndexType, IndexType, IndexType, const Foam::UList<IndexType> &,
               const List &,   const CombineOp &);
        IndexType delta(void) const;
        const_reference lowSum(void) const;
        const_reference highSum(void) const;
        void add(const_reference, const_reference, const CombineOp &);
        void add(const Foam::UList<IndexType> &, const List &, const CombineOp &);
    }

    -exec p count
    $34 = {<Foam::Field<double>> = {<Foam::FieldBase> = {<Foam::refCount> = {count_ = 0},   static typeName = 0x7ffff7d4e3de "Field", static allowConstructFromLargerSize = false},   <Foam::List<double>> = {<Foam::UList<double>> = {size_ = 10, v_ = 0x472490}, <No data     fields>}, <No data fields>}, min_ = 0, max_ = 1, delta_ = 0.10000000000000001, lowSum_ =    0, highSum_ = 0}
    ```

    So we can see that `BinSum` is a templated class inheriting from `List`. Let's take a look at the `UList` of the `BinSum` variable `count`.

    ```c++
    -exec p *count.v_@count.size_
    $36 = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0}

    -exec p *sum.v_@sum.size_
    $40 = {0, 0, 0, 0, 0, 0, 0, 0, 0, 0}

    // Looping once
    -exec p *count.v_@count.size_
    $42 = {0, 0, 0, 1, 0, 0, 0, 0, 0, 0}

    -exec p *sum.v_@sum.size_
    $43 = {0, 0, 0, 0.35372820284499129, 0, 0, 0, 0, 0, 0}

    // Looping twice
    -exec p *count.v_@count.size_
    $44 = {0, 0, 1, 1, 0, 0, 0, 0, 0, 0}

    -exec p *sum.v_@sum.size_
    $45 = {0, 0, 0.26022200134234486, 0.35372820284499129, 0, 0, 0, 0, 0, 0}
    ```

    Taking a look at the add function by right clicking it and going to definition.

    ```c++
    template<class IndexType, class List, class CombineOp>
    void Foam::BinSum<IndexType, List, CombineOp>::add
    (
        const IndexType& indexVal,
        const typename List::const_reference val,
        const CombineOp& cop
    )
    {
        if (indexVal < min_) // Place a breakpoint here
        {
            cop(lowSum_, val);
        }
        else if (indexVal >= max_)
        {
            cop(highSum_, val);
        }
        else
        {
            label index = (indexVal-min_)/delta_;
            cop(this->operator[](index), val);  //continue until here
        }
    }
    ```

    Place a breakpoint as instructed above and run the loop a third time. This will break in the `add` function from where we continue until the final line (after `index`has been initialized, but before we exit). In the vs-code Debug console type

    ```c++
    -exec ptype indexVal
    type = const double &

    indexVal
    0.77678995132180573
    indexVal-min_
    0.77678995132180573
    (indexVal-min_)/delta_
    7.7678995132180573
    index
    7
    ```

    So we see that the add function adds the value given by the second argument (`1` or `samples[i]`) to an index in the `count` or `sum` Ulist, where the index is chosen using the mapping `(indexVal-min_)/delta_` which maps the uniformly distributed value $\in[0,1]$ to a value between 0 and 10, which it then casts to an integer. Incidently, note that indexVal cannot be `1` as this would given an `index=10` which would be "out-of-bounds". This is treated by the line `else if (indexVal >= max_)`.

    ```c++

    // After looping three times
    -exec p *count.v_@count.size_
    $49 = {0, 0, 1, 1, 0, 0, 0, 1, 0, 0}

    -exec p *sum.v_@sum.size_
    $   50 = {0, 0, 0.26022200134234486, 0.35372820284499129, 0, 0, 0, 0.77678995132180573, 0, 0}
    ```

    At this point we are more or less finished with examining this part of the code. However let's take a look at the `forAll` macro on line 63 by typing `-exec info macro forAll`

    ```c
    Defined at /opt/OpenFOAM/OpenFOAM-v1912/src/OpenFOAM/lnInclude/stdFoam.H:290
    included at /opt/OpenFOAM/OpenFOAM-v1912/src/OpenFOAM/lnInclude/UList.H:54
    included at /opt/OpenFOAM/OpenFOAM-v1912/src/OpenFOAM/lnInclude/List.H:46
    included at /home/tike/OpenFOAM/tike-v1912/run/test/BinSum/Test-BinSum.C:34
    #define forAll(list, i) for (Foam::label i=0; i<(list).size(); ++i)
    ```

    Let's see how that macro looks like with some arguments, try `-exec macro expand forAll(samples, i)`

    ```c++
    expands to: for (Foam::label i=0; i<(samples).size(); ++i)
    ```

    The only bit remaining is to assess the `Foam::plusEqOp<double>`. Nonetheless, the operation being performed is by now quite self-evident. The program creates a random set of 10000000 numbers uniformly distributed between 0 and 1 then separates them into 10 uniformly spaced bins based on their value, where count contains a list of the number of samples in each bin and sum contains the corresponding sum of values in each bin. At the end the values are printed to screen

    ```c++
    Info<< "sum    : " << sum << endl;
    Info<< "count  : " << count << endl;
    Info<< "average: " << sum/count << endl;

    /* This printed the following in the terminal when testing
    sum    : 10(50074 150021 250020 349644 450563 549459 649356
                750046 850374 949619)
    count  : 10(1.0015e+06 1.00004e+06 1.00012e+06 998967
                1.00117e+06 999029 998997 1.0001e+06
                1.00042e+06 999650)
    average: 10(0.0499987 0.150014 0.249991 0.350005 0.450037
                0.549993 0.650008 0.749971 0.850012 0.949951)
    */

    ```

12. Before exiting this tutorial, let's take a look at the shear number of macros in OpenFOAM by typing `-exec info macros`. This will take some time to run, but eventually we can view the [output](GDBMacros.md).
