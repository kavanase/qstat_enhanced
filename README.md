# Enhanced `qstat`
Formatted qstat output to show relevant job properties
(such as CPUs, elapsed runtime, remaining runtime etc.)
for efficient HPC job management.

The current implementation can be used with
UCL(-managed) HPC systems (Myriad, Grace, Kathleen,
Thomas), via `qstat_enhanced`, or with Imperial
College's HPC systems (CX1/CX2), via
`qstat_with_cpus_elaptime`.

Could likely be very simply modified to work with other HPC systems too.

## Implementation
### UCL HPC (Sun Grid Engine (SGE) Job Scheduler)
```
$ q

Job ID   Priority           Job Name           Status   CPUs   State    Elap_Time    Remaining
-------- ---------- -------------------------- -------- ------ ------- ------------- -----------
42656    3.50000    CdonTe222Antisitea0.345    running  240    r       6.35 hours    41.65 hours
42633    0.00000    CdonTe222Antisitea0.345    pending  240    hqw     Fuck all pal  24.00 hours
```

### ICL HPC (PBS Job Scheduler)
```
$ q

Job ID     Class          Job Name        Status   CPUs    RTime     Comment
-------- ----------- -------------------- -------- ------ --------- -----------
1271971  Large       222kCdVacfrmibrion2  Running  576    07:36:55  finishing on Mon Apr 06 at 03:51
1274343  Capability  222kCdTeSinglePoint  Queued   2016   Fuck all  starting by Sat Apr  4 17:05
```

## Installation
```bash
git clone https://github.com/kavanase/qstat_enhanced.git
# For UCL HPC:
pip install xmltodict # qstat_enhanced dependency
echo "alias q=/path/to/qstat_enhanced/bin/qstat_enhanced" >> ~/.bashrc
# or for ICL HPC:
echo "alias q=/path/to/qstat_enhanced/bin/qstat_with_cpus_elaptime" >> ~/.bashrc
```

## Job History
### UCL HPC
Thankfully, `qstat` job history is maintained on all UCL HPC systems, via the
`jobhist` command. I set the alias `jh` in my `bashrc` file:
```bash
alias jh="jobhist --info='fstime,fetime,job_number,job_name,slots'"
```
as the standard output of `jobhist` gives a lot of irrelevant info
(such as `HOSTNAME` (internal name for node the job ran on),
`OWNER` (your username) etc.), and doesn't tell you how many CPUs the jobs used.

### Imperial HPC (CX1/CX2)
As of February 2020, job history is no longer maintained by `qstat`
(previously accessible with `qstat -x`).
As an alternative, in order to keep track of my jobs, I add these lines of code
to my PBS jobscripts:
```bash
start=$(date +%s)
mpiexec my_program
cpus=$( find . -maxdepth 1 -name "vasp_out*" | xargs ls -t | head -1 | xargs head | awk '/ranks allocated/{print $3}' )
runtime_sec=$(($(date +%s) - start))
runtime=$( date -d@$runtime_sec -u +%H:%M:%S )
printf "%-13s %-13s %-4s %-9s %-50s \n" "$( echo ${PBS_JOBID} | cut -c -7)" \
"$( date "+%a %H:%M" )" "$cpus" "$runtime" "$( echo ${PBS_O_WORKDIR} | cut -c 43- )" >> ~/job_log
```
and this to my `.bashrc`, to give me an update on any completed jobs:
```bash
echo " Job Num     Finish Time    CPUs  Runtime                    Working Directory"
echo "--------- ----------------- ---- --------- --------------------------------------------------"
tail -5 ~/job_log
 ```

### Acknowledgement
Several of the functions in `qstat_enhanced` are based on the [qstat](https://github.com/relleums/qstat) repository by [relleums](https://github.com/relleums).
