These notes are based mainly on [this](https://stackoverflow.com/questions/46421663/what-are-jupyter-notebook-checkpoint-files-for) Stack Overflow page.

When working with Jupyter, I have observed that besides the actual notebook file, (e.g. `test.ipynb`) we sometimes see a checkpoint `.ipynb_checkpoints` directory. For instance: 

![](_attachments/Screenshot%202023-06-29%20at%2020.50.45.png)

Inside is another `.ipynb` file:

![](_attachments/Screenshot%202023-06-29%20at%2020.52.19.png)

What is the purpose of this files? Under what conditions does it appear? How can I make use of it?

It turns out that, whenever we create a notebook file, Jupyter creates a checkpoint file and stores it in the `.ipynb_checkpoints/` folder. This folder is kept in the same directory as the notebook file.

Whenever you *manually* save your progress in the notebook file (e.g. by hitting `cmd + s`), then Jupyter updates this checkpoint file. It is an exact copy of the notebook at that stage.

Auto-saving, on the other hand, updates only the initial `.ipynb` file, not the checkpoint file.

You can revert from the initial `.ipynb` file to a previously saved checkpoint by using the *revert to checkpoint* button. The checkpoint file is then accessed and opened inside Jupyter.

> In summary, the `.ipynb_checkpoints` folder acts as a simple backup for your notebooks. You can exploit it by manually saving the notebook at points you deem useful.