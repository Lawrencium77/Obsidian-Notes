These are just a few notes I made about moving jobs to the foreground/background in Linux. They helped me make some Ankis.

List of bits:

> To pause a job, we type `ctrl-Z`.

> To resume it in the foreground, we simply type `fg`. We use `bg` for the background

> To see a list of running jobs, we type `jobs`.

> We can start a job running in the background by appending an ampersand, e.g. `sleep 20 &`.

> If we begin and pause multiple jobs, typing `bg` will then resume them in the background in a FIFO fashion:

```zsh
❯ sleep 20
^Z
[1]  + 6357 suspended  sleep 20
❯ sleep 20
^Z
[2]  + 6366 suspended  sleep 20
❯ bg
[2]  - 6366 continued  sleep 20
❯ bg
[1]  - 6357 continued  sleep 20

```

> `fg`, `bg`, and `jobs` are a separate set of commands to `ps`.

> To resume a specific job, we do need a `%` symbol prior to the job number. E.g: 

```zsh
❯ jobs
[1]  - suspended  sleep 20
[2]  + suspended  sleep 20
❯ bg %1
[1]  - 6390 continued  sleep 20
```