SparseDistance
==============

Efficiently generate sparse graph adjacency matrices using tensorflow, including gradient propagation and minibatches.

 - Input: a set of elements with features, shape `(N_batch, N_elem, N_feat)`
 - Output: a sparse adjacency matrix with shape `(N_batch, N_elem, N_elem)`, the elements of which can be differentiated with respect to the input
 - Parameters: bin size M, number of neighbors K

1. Each element in the input set is assigned into a bin using locality-sensitive hashing
2. In each bin, we run a dense k-nearest neighbors algorithm
3. The neighbors found in each bin are propagated to the final sparse adjacency matrix


```python
from sparsedistance.models import SparseHashedNNDistance
from sparsedistance.utils import sparse_dense_matmult_batch, pairwise_dist

num_batches = 10
num_points_per_batch = 1000
num_features = 32

X = np.array(np.random.randn(num_batches, num_points_per_batch, num_features), dtype=np.float32)
y = np.array(np.random.randn(num_batches, num_points_per_batch, ), dtype=np.float32)

#show that we can take a gradient of stuff with respect to the distance matrix values (but not indices!)
dense_transform = tf.keras.layers.Dense(128)
dm_layer = SparseHashedNNDistance()

with tf.GradientTape(persistent=True) as g:
    X_transformed = dense_transform(X)
    dm = dm_layer(X_transformed)

    ret = sparse_dense_matmult_batch(dm, X)

    #reduce the output to a single scalar, just for demonstration purposes
    ret = tf.reduce_sum(ret)

grad = g.gradient(ret, dense_transform.weights)
```

<p float="left">
  <img src="images/graph_noedge.png" alt="Input set without edges" width="300"/>
  <img src="images/graph.png" alt="Genetated graph with edges" width="300"/>
</p>

<p float="left">
  <img src="images/test.png" alt="Generated adjacency matrix" width="300"/>
  <img src="images/timing.png" alt="Scaling of the complexity with input size" width="300"/>
</p>