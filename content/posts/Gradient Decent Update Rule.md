---
title: "Gradient Decent Update Rule"
date: 2021-03-08T21:32:56+08:00
draft: false
---

### Gradient descent update rule

$$ W^{[l]} = W^{[l]} - \alpha \text{ } dW^{[l]}\tag{1} $$

$$ b^{[l]} = b^{[l]} - \alpha \text{ } db^{[l]}\tag{2} $$

> where L is the number of layers and αα is the learning rate. All parameters should be stored in the `parameters` dictionary.
>
> Note that the iterator `l` starts at **0** in the `for loop` while the first parameters are `W[1]` and `b[1]`.
>
> You need to shift `l` to `l+1` when coding.

```python
def update_parameters_with_gd(parameters, grads, learning_rate):
    """
    Update parameters using one step of gradient descent

    Arguments:
    parameters -- python dictionary containing your parameters to be updated:
                    parameters['W' + str(l)] = Wl
                    parameters['b' + str(l)] = bl
    grads -- python dictionary containing your gradients to update each parameters:
                    grads['dW' + str(l)] = dWl
                    grads['db' + str(l)] = dbl
    learning_rate -- the learning rate, scalar.

    Returns:
    parameters -- python dictionary containing your updated parameters
    """

    L = len(parameters) // 2 # number of layers in the neural networks

    # Update rule for each parameter
    for l in range(L):
        ### START CODE HERE ### (approx. 2 lines)
        parameters["W" + str(l+1)] = parameters["W" + str(l+1)] - learning_rate * grads["dW" + str(l + 1)]
        parameters["b" + str(l+1)] = parameters["b" + str(l+1)] - learning_rate * grads["db" + str(l + 1)]
        ### END CODE HERE ###

    return parameters
```

---

### Different between GD & SGD

- **(Batch) Gradient Descent**

```python
X = data_input
Y = labels
parameters = initialize_parameters(layers_dims)
for i in range(0, num_iterations):
    # Forward propagation
    a, caches = forward_propagation(X, parameters)
    # Compute cost.
    cost = compute_cost(a, Y)
    # Backward propagation.
    grads = backward_propagation(a, caches, parameters)
    # Update parameters.
    parameters = update_parameters(parameters, grads)
```

- **Stochastic Gradient Descent**:

```python
X = data_input
Y = labels
parameters = initialize_parameters(layers_dims)
for i in range(0, num_iterations):
    for j in range(0, m):
        # Forward propagation
        a, caches = forward_propagation(X[:,j], parameters)
        # Compute cost
        cost = compute_cost(a, Y[:,j])
        # Backward propagation
        grads = backward_propagation(a, caches, parameters)
        # Update parameters.
        parameters = update_parameters(parameters, grads)
```

In Stochastic Gradient Descent, you use only 1 training example before updating the gradients. When the training set is large, SGD can be faster. But the parameters will "oscillate" toward the minimum rather than converge smoothly. Here is an illustration of this:

{% asset_img 1550575038653.png %}

> "+" denotes a minimum of the cost. SGD leads to many oscillations to reach convergence. But each step is a lot faster to compute for SGD than for GD, as it uses only one training example (vs. the whole batch for GD).

**Note** also that implementing SGD requires 3 for-loops in total:

1. Over the number of iterations
2. Over the $m$ training examples
3. Over the layers (to update all parameters, from $(W^{[1]},b^{[1]})$ to $(W^{[L]},b^{[L]})$)

---

### Mini-Batch Gradient descent

2 Steps

- **Shuffle**

{% asset_img 1550575200308.png %}

- **Partition**: Partition the shuffled (X, Y) into `mini_batch_size`

{% asset_img 1550575213693.png %}

Note that the last mini-batch might end up smaller than `mini_batch_size=64`. Let $$ \lfloor s \rfloor $$ represents $$ s $$ rounded down to the nearest integer (this is `math.floor(s)` in Python). If the total number of examples is not a multiple of `mini_batch_size=64` then there will be $$\lfloor \frac{m}{mini\_batch\_size}\rfloor$$ mini-batches with a full 64 examples, and the number of examples in the final mini-batch will be $$ m-mini\_batch\_size \times \lfloor \frac{m}{mini\_batch\_size}\rfloor $$

```python
num_complete_minibatches = math.floor(m/mini_batch_size) # number of mini batches of size mini_batch_size in your partitionning
    for k in range(0, num_complete_minibatches):
        ### START CODE HERE ### (approx. 2 lines)
        mini_batch_X = shuffled_X[:, k * mini_batch_size :(k + 1) * mini_batch_size]
        mini_batch_Y = shuffled_Y[:, k * mini_batch_size :(k + 1) * mini_batch_size]
        ### END CODE HERE ###
        mini_batch = (mini_batch_X, mini_batch_Y)
        mini_batches.append(mini_batch)

    # Handling the end case (last mini-batch < mini_batch_size)
    if m % mini_batch_size != 0:
        ### START CODE HERE ### (approx. 2 lines)
        mini_batch_X = shuffled_X[:,num_complete_minibatches * mini_batch_size:]
        mini_batch_Y = shuffled_Y[:,num_complete_minibatches * mini_batch_size:]
        ### END CODE HERE ###
        mini_batch = (mini_batch_X, mini_batch_Y)
        mini_batches.append(mini_batch)
```

---

### Momentum

- Initialize

```python
for l in range(L):
        v["dW" + str(l+1)] = np.zeros_like(parameters["W"+str(l+1)])
        v["db" + str(l+1)] = np.zeros_like(parameters["b"+str(l+1)])
```

- Update

$$
v_{dW^{[l]}} = \beta v_{dW^{[l]}} + (1 - \beta) dW^{[l]} \\
W^{[l]} = W^{[l]} - \alpha v_{dW^{[l]}}
\end{cases}\tag{1}$$
$$

v\\*{db^{[l]}} = \beta v\\*{db^{[l]}} + (1 - \beta) db^{[l]} \\
b^{[l]} = b^{[l]} - \alpha v\\\_{db^{[l]}}
\end{cases}\tag{2}\\\$\$

> where L is the number of layers, $\beta$ is the momentum and $\alpha$ is the learning rate. All parameters should be stored in the `parameters` dictionary. Note that the iterator `l` starts at 0 in the `for` loop while the first parameters are $W^{[1]}$ and $b^{[1]}$ (that's a "one" on the superscript). So you will need to shift `l` to `l+1` when coding.

```python
for l in range(L):
        # compute velocities
        v["dW" + str(l+1)] = beta * v["dW" + str(l + 1)] + (1 - beta) * grads['dW' + str(l + 1)]
        v["db" + str(l+1)] = beta * v["db" + str(l + 1)] + (1 - beta) * grads['db' + str(l + 1)]
        # update parameters
        parameters["W" + str(l+1)] = parameters["W" + str(l + 1)] - learning_rate * v["dW"+str(l + 1)]
        parameters["b" + str(l+1)] = parameters["b" + str(l + 1)] - learning_rate * v["db"+str(l + 1)]
```

---

### Adam

#### Synopsis

1. It calculates an exponentially weighted average of past gradients, and stores it in variables $v$ (before bias correction) and $v^{corrected}$ (with bias correction).

2. It calculates an exponentially weighted average of the squares of the past gradients, and stores it in variables $s$ (before bias correction) and $s^{corrected}$ (with bias correction).

3. It updates parameters in a direction based on combining information from "1" and "2".

#### Update


$$ v_{dW} = \beta_1 v_{dW} + (1 - \beta_1) dW  $$
$$ v_{db} = \beta_1 v_{db} + (1 - \beta_1) db $$
$$ s_{dW} = \beta_2 s_{dW} + (1 - \beta_2) (dW)^2 $$
$$ s_{db} = \beta_2 s_{db} + (1 - \beta_2) (db)^2 $$

---

$$ v^{corrected}{dW} = \frac{v_{dW}}{1 - (\beta*1)^t} $$
$$ v^{corrected}* {db} = \frac{v*{db}}{1 - (\beta_1)^t} $$
$$ s^{corrected}* {dW} = \frac{s*{dW}}{1 - (\beta_1)^t} $$
$$ s^{corrected}* {db} = \frac{s\_{db}}{1 - (\beta_2)^t} $$

---

$$ W = W - \alpha \frac{v^{corrected}_{dW}}{\sqrt{s^{corrected}_{dW} + \varepsilon}} $$
$$ b = b - \alpha \frac{v^{corrected}_{db}}{\sqrt{s^{corrected}_{db} + \varepsilon}} $$

---

where:

- t counts the number of steps taken of Adam
- L is the number of layers
- $\beta_1$ and $\beta_2$ are hyperparameters that control the two exponentially weighted averages.
- $ \alpha $ is the learning rate
- $ \varepsilon $ is a very small number to avoid dividing by zero

```python
for l in range(L):
        # Moving average of the gradients. Inputs: "v, grads, beta1". Output: "v".
        v["dW" + str(l+1)] = beta1 * v["dW" + str(l + 1)] + (1 - beta1) * grads['dW' + str(l + 1)]
        v["db" + str(l+1)] = beta1 * v["db" + str(l + 1)] + (1 - beta1) * grads['db' + str(l + 1)]

        # Compute bias-corrected first moment estimate. Inputs: "v, beta1, t". Output: "v_corrected".
        v_corrected["dW" + str(l+1)] = v["dW" + str(l+1)]/(1 - np.power(beta1, t))
        v_corrected["db" + str(l+1)] = v["db" + str(l+1)]/(1 - np.power(beta1, t))

        # Moving average of the squared gradients. Inputs: "s, grads, beta2". Output: "s".
        s["dW" + str(l+1)] = beta2 * s["dW" + str(l + 1)] + (1 - beta2) * np.power(grads['dW' + str(l + 1)], 2)
        s["db" + str(l+1)] = beta2 * s["db" + str(l + 1)] + (1 - beta2) * np.power(grads['db' + str(l + 1)], 2)

        # Compute bias-corrected second raw moment estimate. Inputs: "s, beta2, t". Output: "s_corrected".
        s_corrected["dW" + str(l+1)] = s["dW" + str(l+1)]/(1 - np.power(beta2, t))
        s_corrected["db" + str(l+1)] = s["db" + str(l+1)]/(1 - np.power(beta2, t))

        # Update parameters. Inputs: "parameters, learning_rate, v_corrected, s_corrected, epsilon". Output: "parameters".
        parameters["W" + str(l+1)] = parameters["W" + str(l + 1)] - learning_rate * v_corrected["dW" + str(l + 1)] / np.sqrt(s_corrected["dW" + str(l + 1)] + epsilon)
        parameters["b" + str(l+1)] = parameters["b" + str(l + 1)] - learning_rate * v_corrected["db" + str(l + 1)] / np.sqrt(s_corrected["db" + str(l + 1)] + epsilon)
```
$$
