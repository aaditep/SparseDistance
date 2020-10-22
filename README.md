SparseDistance
==============

Efficiently generate sparse graph adjacency matrices using tensorflow, including gradient propagation and minibatches.

 - *Input*: a set of elements with features, `shape=(N_batch, N_elem, N_feat)`, possibly in minibatches for efficient training (e.g. each graph is a different minibatch)
 - *Output*: a sparse adjacency matrix `shape=(N_batch, N_elem, N_elem)`, the elements of which can be differentiated with respect to the input
 - *Parameters*: bin size M, number of neighbors K, LSH codebook size (maximum number of bins) L

The input data is divided into equal-sized bins based on a locality sensitive hashing (LSH). In each bin, we run a dense k-nearest-neighbors and update the final sparse adjacency matrix. 
The maximum input size is determined by the pre-generated LSH codebook size, which is based on random rotations. Since the bin size is much smaller than the input size, the k-nearest-neighbors evaluation is efficient.
The input features to the LSH hashing are learnable, so the binning can adapt to the problem based on gradient descent.

```python
from sparsedistance.models import SparseHashedNNDistance
from sparsedistance.utils import sparse_dense_matmult_batch

num_batches = 10
num_points_per_batch = 1000
num_features = 32

X = np.array(np.random.randn(num_batches, num_points_per_batch, num_features), dtype=np.float32)
y = np.array(np.random.randn(num_batches, num_points_per_batch, ), dtype=np.float32)

#show that we can take a gradient of stuff with respect to the distance matrix values (but not indices!)
dense_transform = tf.keras.layers.Dense(128)
dm_layer = SparseHashedNNDistance(max_num_bins=200, bin_size=500, num_neighbors=5)

with tf.GradientTape(persistent=True) as g:
    X_transformed = dense_transform(X)
    dm = dm_layer(X_transformed)

    ret = sparse_dense_matmult_batch(dm, X)

    #reduce the output to a single scalar, just for demonstration purposes
    ret = tf.reduce_sum(ret)

grad = g.gradient(ret, dense_transform.weights)
```

On the following images, you see the input set on the left and the learned graph structure (edges) on the right for a toy clustering problem.
<p float="left">
  <img src="images/graph_noedge.png" alt="Input set without edges" width="300"/>
  <img src="images/graph.png" alt="Genetated graph with edges" width="300"/>
</p>

Here, we show the learned distance matrix on the left and the scaling of the training time on the right.
<p float="left">
  <img src="images/dm.png" alt="Generated adjacency matrix" width="300"/>
  <img src="images/timing.png" alt="Scaling of the complexity with input size" width="300"/>
</p>

Features:
 - [x] Work on a modest GPU (e.g. 2060S) or a CPU
 - [x] Uses only native TF 2.x operations, no compilation needed
 - [x] Fast evaluation and efficient memory use

Based on the Reformer [1] and GravNet [2] papers.

 - [1] https://arxiv.org/abs/2001.04451
 - [2] https://arxiv.org/abs/1902.07987