These notes are based on Section 3 of the [Megatron LM](https://arxiv.org/pdf/1909.08053.pdf) paper. They aim to explain exactly how tensor parallel works.

```toc
```

## MLP

Let's first understand how MLP blocks work. The **first part** of the block is:

$$Y=\textrm{GeLU}(XA)$$

where $X$ is the input and $A$ is the weight matrix.
One option to parallelise the GEMM is to split the $A$ along its rows and $X$ along its columns:

$$X=[X_1,X_2],A=\begin{bmatrix}  
A_1 \\  
A_2   
\end{bmatrix}$$
E.g. the first shard looks like:

![](_attachments/Screenshot%202023-04-22%20at%2020.00.30.png)

This means we compute half the sum required for each element of the output. In other words:

$$Y=\textrm{GeLU}(X_1A_1+X_2A_2)$$

Since $\textrm{GeLU}$ is nonlinear, this requires a synchronization point before the $\textrm{GeLU}$ function. 

Another option is to split $A$ along it columns $A=[A_1,A_2]$, and to not split $X$ at all:

![](_attachments/Screenshot%202023-04-22%20at%2020.01.06.png)

This allows the $\textrm{GeLU}$ to be independently applied to the output of each partitioned GEMM:

$$[Y_1,Y_2]=[\textrm{GeLU}(XA_1),\textrm{GeLU}(XA_2)]$$

This is beneficial since it removes a synchronisation point.
We split the second GEMM along its rows so that it takes the output of the $\textrm{GeLU}$ directly without requiring any communication:

![](_attachments/Screenshot%202023-04-23%20at%2017.12.25.png)

The output of this second GEMM is then reduced (specifically, summed) across GPUs before entering the dropout layer:

![](_attachments/Screenshot%202023-04-22%20at%2020.06.04.png)
$f$ and $g$ are conjugate; $f$ is an identity in the forward pass and an AllReduce in the backwards. $g$ is an AllReduce in the forwards, and an identity in the backwards.

Ultimately, the benefit of this approach is that both GEMMs in the MLP block are split across GPUs. Only a single AllReduce operation is required in the forward, and one in the backward too.

## Attention Block
For the attention block, we exploit inherent parallelism in multiheaded attention.
We partition the QKV GEMMs in a column parallel fashion such the matmul corresponding to each attention head is done locally on one GPU.
For example, for four heads:

![](_attachments/Screenshot%202023-04-23%20at%2016.49.30.png)

The final projection GEMM is parallelised along rows, just like the second GEMM in the [MLP](#MLP) layer.  This means it can take the output from each head directly. For both the MLP and Attention blocks, this approach fuses groups of two GEMMs, eliminating synchronization points between them.

Overall, the attention block looks like:

![](_attachments/Screenshot%202023-04-23%20at%2016.52.59.png)

Again, we only need one all-reduce in the forwards pass, and one all-reduce in the backwards pass.

## Overall
In total, there are a total of 4 communication operations in the forward and backward pass of a single model parallel transformer layer:

![](_attachments/Screenshot%202023-04-23%20at%2016.54.21.png)

The tensor parallel approach can be characterised as aiming to reduce communication and keep GPUs compute bound. 
Rather than having one GPU compute part of the dropout, layer norm, or residual connections and broadcast the results to other GPUs, we duplicate the computation across GPUs. 
To optimise the model we allow each model parallel worker to optimise its own set of parameters. Since all values are local to a GPU, there is no need for communicating updated parameter values.







