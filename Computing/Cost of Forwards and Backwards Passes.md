[I have seen it described](https://ai.stackexchange.com/questions/38085/why-should-one-expect-the-backward-pass-to-take-twice-as-long-as-the-forward-pas) that the computational cost of the forwards pass is twice as large as that of the forwards pass. Why is this?

[This LessWrong comment](https://www.lesswrong.com/posts/fnjKpBoWJXcSDwhZk/what-s-the-backward-forward-flop-ratio-for-neural-networks?commentId=QTaEstMexB4DAc8wj) suggests thinking about it in terms of **connections** between layers. We have:

* One FLOP per connection in the forward pass
* One FLOP per connection in the backward gradient pass
* One FLOP per connection for the weight update calculation

Hence the ratio of compute costs in the backwards and forwards passes is **2:1**.

