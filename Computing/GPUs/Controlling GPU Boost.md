The information used to make these notes was obtained form the 5 different websites: [1](https://www.microway.com/hpc-tech-tips/nvidia-smi_control-your-gpus/), [2](https://developer.nvidia.com/blog/increase-performance-gpu-boost-k80-autoboost/), [3](https://docs.amazonaws.cn/en_us/AWSEC2/latest/UserGuide/optimize_gpu.html), [4](https://nvidia.custhelp.com/app/answers/detail/a_id/3751/~/useful-nvidia-smi-queries), [5](https://stackoverflow.com/questions/64701751/can-i-fix-my-gpu-clock-rate-to-ensure-consistent-profiling-results)

> [!INFO]
> In order to run a lot of the commands list here, it is necessary to have root permissions. This can be done by running `sudo su willw` before SSHing into a box. 
```toc
```
## Querying Clock Speed
When on a box, we can show the current  clock settings with `nvidia-smi -q -i 0 -d CLOCK` (for GPU id 0). This produces something like[^fn1]:

```C++
==============NVSMI LOG==============

Timestamp                                 : Fri Sep  9 10:16:08 2022
Driver Version                            : 510.73.08
CUDA Version                              : 11.6

Attached GPUs                             : 7
GPU 00000000:07:00.0
    Clocks
        Graphics                          : 1365 MHz
        SM                                : 1365 MHz
        Memory                            : 1593 MHz
        Video                             : 1230 MHz
    Applications Clocks
        Graphics                          : 1155 MHz
        Memory                            : 1593 MHz
    Default Applications Clocks
        Graphics                          : 1155 MHz
        Memory                            : 1593 MHz
    Max Clocks
        Graphics                          : 1410 MHz
        SM                                : 1410 MHz
        Memory                            : 1593 MHz
        Video                             : 1290 MHz
    Max Customer Boost Clocks
        Graphics                          : 1410 MHz
    SM Clock Samples
        Duration                          : Not Found
        Number of Samples                 : Not Found
        Max                               : Not Found
        Min                               : Not Found
        Avg                               : Not Found
    Memory Clock Samples
        Duration                          : Not Found
        Number of Samples                 : Not Found
        Max                               : Not Found
        Min                               : Not Found
        Avg                               : Not Found
    Clock Policy
        Auto Boost                        : N/A
        Auto Boost Default                : N/A
```

The sections we really care about "Clocks" (showing current clock speed), and "Max Clocks" (maximum possible clock speed). 

A listing of available clock speeds can be obtained with `nvidia-smi -q -i 0 -d SUPPORTED_CLOCKS`:

```C++
==============NVSMI LOG==============

Timestamp                                 : Fri Sep  9 10:23:21 2022
Driver Version                            : 510.73.08
CUDA Version                              : 11.6

Attached GPUs                             : 7
GPU 00000000:07:00.0
    Supported Clocks
        Memory                            : 1593 MHz
            Graphics                      : 1410 MHz
            Graphics                      : 1395 MHz
            Graphics                      : 1380 MHz
            Graphics                      : 1365 MHz
            Graphics                      : 1350 MHz
					        .
						    .
						    .
			Graphics                      : 225 MHz
            Graphics                      : 210 MHz
```

From this, we can see that the only clock that varies speed is the one labelled "Graphics".

To see if clock speed does vary between GPUs during training, I measured the clock speeds on b11 (that was training koala for 850k steps at the time). The results confirm that clock speed does change: ^f32c95

```C++
❯ nvidia-smi -q -d CLOCK

==============NVSMI LOG==============

Timestamp                                 : Fri Sep  9 10:29:25 2022
Driver Version                            : 510.85.02
CUDA Version                              : 11.6

Attached GPUs                             : 8
GPU 00000000:07:00.0
    Clocks
        Graphics                          : 1350 MHz
        SM                                : 1350 MHz
        Memory                            : 1593 MHz
        Video                             : 1215 MHz
							.
							.
							.
GPU 00000000:0A:00.0
    Clocks
        Graphics                          : 1365 MHz
        SM                                : 1365 MHz
        Memory                            : 1593 MHz
        Video                             : 1245 MHz
							.
							.
							.
GPU 00000000:49:00.0
    Clocks
        Graphics                          : 1320 MHz
        SM                                : 1320 MHz
        Memory                            : 1593 MHz
        Video                             : 1200 MHz
					        .
					        .
					        .
```

Whilst not seen here, I have seen clock speeds that reached the maximum possible value (1410).

The final point to note is that the idle clock speed is the minimum supported block speed (210 in this case):

```C++
==============NVSMI LOG==============

Timestamp                                 : Fri Sep  9 10:47:05 2022
Driver Version                            : 510.73.08
CUDA Version                              : 11.6

Attached GPUs                             : 7
GPU 00000000:07:00.0
    Clocks
        Graphics                          : 210 MHz
        SM                                : 210 MHz
        Memory                            : 1593 MHz
        Video                             : 585 MHz

```

## Changing Clock Speed
Ideally, we could run a command that disables GPU boost. However, I have not been able to find one. The main candidate is `sudo nvidia-smi --auto-boost-default=0`, but this throws the following error:

```C++
Enabling/disabling default auto boosted clocks is not supported for GPU: 00000000:C2:00.0.
Treating as warning and moving on.
All done.
```

I have also tried `sudo nvidia-smi --lock-gpu-clocks`, just like they do in the. 

Instead, it is possible to manually set the clock speed with a command like `sudo nvidia-smi -ac 1593,225`. The first number here sets the memory clock. The second sets the graphics clock. The graphics clock must be set to a supported clock speed.  Clock speed can then be reset to the default with `sudo nvidia-smi  -rac`. We could also use `sudo nvidia-smi --lock-gpu-clocks=225,225` to do this, like in the  [AlphaTensor repo](https://github.com/deepmind/alphatensor/blob/main/benchmarking/run_gpu_benchmark.py#L61).

The application clocks setting is a recommendation. If the GPU cannot safely run at the selected clocks, for example due to thermal or power reasons, it will dynamically lower the clocks. This means that setting the clock speed doesn't necessarily mean it is fixed.

However, I believe that setting the clock speed to a low value (significantly lower than the [minimum seen during a typical training run](#^f32c95)) can prevent GPU boost from causing variable clock speeds in practice. GPU boost will never raise the clock speed above the value set, and is very unlikely to lower it.

I found a diagram that illustrates this [website #2](https://developer.nvidia.com/blog/increase-performance-gpu-boost-k80-autoboost/):

![](_attachments/Screenshot%202022-09-09%20at%2011.25.40.png)

The RHS shows a GPU without a fixed clock speed, and autoboost. The LHS shows a GPU with fixed clock speed.

As an example of how clock speeds affect performance, I timed a whole `InvariantTransformer()` object, with warmup and train steps both set to 10. 

Clock rate set to 210 (the minimum):

![](_attachments/Screenshot%202022-09-09%20at%2010.01.57.png)

Clock rate set to default (i.e. allowed to reach the maximum when possible):

![](_attachments/Screenshot%202022-09-09%20at%2010.02.51.png)

Lowering the clock speed significantly decreases throughput, as expected.

[^fn1]: These numbers were obtained from a GPU that was active at the time of measurement.


To add:

* Lock **GPU** clocks
* OpenAI set clock speed to 1350. Tradeoff between speed and variance?