# CaseRunner

A way to run a list of GenX cases as separate batch jobs, or locally in sequence as one job.

# Invocation
Ensure that the julia binary is available. (On a cluster, ensure that the Julia module is loaded.)

```
> module load julia
```

Starting from the CaseRunner directory, run the case runner script.
```
> julia caserunner.jl
```

# How the template scheme works

* One or more of the entries in the various .csv files are replaced by "special key" strings which take the form `__SPECIAL_Abcd__`, `__SPECIAL_xyz__`.
* A csv file called `replacements.csv` has rows which correspond to a user-given case number and replacements for each of the special fields.
* One special key field in the template folder corresponds to exactly one column in the replacements file. No duplicates.
* The program will warn you if you have duplicates or incomplete matching of special keys with replacement files.

## Details of the special key string
* The string must start with `__SPECIAL_` and end with `__`.
* Between the start and end groups there must be a key which does *not* contain an underscore.

Good:
`__SPECIAL_A__`, `__SPECIAL_cow1234__`.
Bad:
`__SPECIAL__`
`__SPECIAL_THIS_is_BAD__`
`__Akey__`

## An example of replacements.csv and the corresponding templates in files

`replacements.csv`
```
Case,  solarcost,    aFloat,  myFuel
1,         10000,      0.02,      NG
2,         20000,      0.04,      NG
50,        30000,      0.06,  biogas
```

`template/Reserves.csv`
```
Reg_Req_Percent_Load, Reg_Req_Percent_VRE, Rsv_Req_Percent_Load, ...
                0.01,  __SPECIAL_aFloat__,                0.033,
```

`template/Generators_data.csv`
```
    Resource,      Inv_Cost_per_MWyr,               Fuel, ...
 natural_gas,                  23045, __SPECIAL_myFuel__,
onshore_wind,                  84030,               None,
    solar_pv,  __SPECIAL_solarcost__,               None,
     battery,                  19034,               None,

```

# How the job scheme works
In batch mode, one slurm batch job is associated with each case.
The batch script is located in the template folder, and is `jobscript.sh`. It is the same for each case.
Cases go into the folder `Cases/case_[n]` where n is the user-supplied (positive integer) case number in the `replacements.csv` file. Cases can be ordered in any fashion.

In sequential mode, cases are run one after the other from the invocing node. This may reduce loading times and allows running cases locally. (This will fail if the local node lacks a solver license.)

## Switching between batch mode and sequential mode
In `caserunner.jl` there's a variable near the top which is set to either `"BATCH"` or `"SEQUENTIAL"`.
Running in sequential mode does not use the `jobscript.sh` file; you will need to ensure that the julia invocation includes the proper environment, such as `julia --project="/my/GenX/" caserunner.jl`.

# Behavior
If you re-run the `caserunner.jl` script it will skip over any cases for which `case_n` folders already exist. It will report that they have been skipped.

This allows the user to add new lines to the `replacements.csv` file: only the new rows will be run. 
This also allows recovery if some runs did not complete, for example due to not enough time allocated in the batch script. If the user manually deletes them they can then modify the batch script in the template folder (to add more time) and re-run. This avoids having to destory or re-run *all* the cases.

# There is no automated job failure detection and re-starting
This feature is not implemented.

# No analysis features
There are no post-run analysis features.
