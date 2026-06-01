---
publish: true
---

### Fuzz

- ***1st argument (DOCIMAGE)*** : name of the docker image
- ***2nd argument (RUNS)*** : number of runs, one isolated Docker container is spawned for each run
- ***3rd argument (SAVETO)*** : path to a folder keeping the results
- ***4th argument (FUZZER)*** : fuzzer name (e.g., aflnet) -- this name must match the name of the fuzzer folder inside the Docker container (e.g., /home/ubuntu/aflnet)
- ***5th argument (OUTDIR)*** : name of the output folder created inside the docker container
- ***6th argument (OPTIONS)*** : all options needed for fuzzing in addition to the standard options written in the target-specific run.sh script
- ***7th argument (TIMEOUT)*** : time for fuzzing in seconds
- ***8th argument (SKIPCOUNT)***: used for calculating coverage over time. e.g., SKIPCOUNT=5 means we run gcovr after every 5 test cases because gcovr takes time and we do not want to run it after every single test case

#### Fuzz-lightftp-aflnet&aflnwe

```bash
cd $PFBENCH
# 统一命名为 results-{protocol}
mkdir results-lightftp
# 12h-> 43200 24h-> 86400
profuzzbench_exec_common.sh lightftp 4 results-lightftp aflnet out-lightftp-aflnet "-P FTP -D 10000 -q 3 -s 3 -E -K" 3600 5 &
profuzzbench_exec_common.sh lightftp 4 results-lightftp aflnwe out-lightftp-aflnwe "-D 10000 -K" 3600 5
# 如果无法运行，则在参数中添加-m none参数
```

