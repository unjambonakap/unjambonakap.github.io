---
layout: post
title:  "hacklu 2018 - Crypto Hard - Escape the grid"
date:   2018-10-18
---

[The challenge](https://arcade.fluxfingers.net/challenges/10), [backup](https://github.com/unjambonakap/ctf/tree/master/hacklu/18/crypto/escape_the_grip)
In case the challenge server is down, some pseudo python code can be seen [below](#resources).

## Intro

Rolling out custom crypto is usually a bad idea, yada yada. Not so much if you take fun into account.  
Here we have a rather complicated cipher:
-  several rounds
-  non linearities
-  a huge keyspace of 385 bits
- an elegant solution

The goal is to recover a given plaintext having access to an oracle returning the result of the encryption of the plaintext with keys of our own choosing.
If that entices you, I encourage to try it.

I will not give a detailed presentation of the encryption function; familiarity assumed (key is **i** in python, plaintext is **key**).

## Meat

The input being split independently per round allows us to change only the parameter **cnt** for the last round, leaving all the previous rounds identical one run to the next.  
 Below the commented code for the last round and the final key whitening.

{% highlight python %}
# The non linear layer. Dunno if it's reversible or not. Don't care.
state = s_layer(state, n)

cnt += i % factorial(n)
i = i / factorial(n)
# Here every bit is unknown but can be chosen to stay
# the same for whichever value of **cnt** in the final round.

# Full control of **cnt** gives us access to every permutation of "matrix".
lin_layer = permutate_matrix(matrix, get_permutation(cnt, n), n)
# No need to look for a backdoor in the fixed matrix.
# Correctly chosing the column permutations allow us to
# break the key in very few trials.
state = matrix_mul(state, lin_layer, n)

# Whitening of the state. Completely useless if we use
# xor differences between output.
# Plus, we figure out the state before whitening, we got the key!
state = add_state(state, key, n)
{% endhighlight %}


Here is the idea that makes the problem solvable:
In the matrix multiplication, swapping two columns in the matrix has the same effect as swapping the corresponding two elements in the vector.
If the swapped bits are the same, result is the same. If not, the result is totally different (assuming random matrix). This potential difference then propagates to the output, unhindered by the whitening.

This gives us a mean to test if two bits are equal in the state right before the final matrix multiplication.
If we do this test smartly n-1=384 times, we are able to partition the bits in two sets, say A and B.  
Then we have to bruteforce a grand total of **2** assignments. Yes, two! (A=0 and B=1, A=1 and B=0).

Finally, with the guessed state, we apply the matrix mul and xor with the ciphertext to retrieve the key and then the flag.
```raw
flag{__xXx_Rasta_best_cipher_in_the_world_xXx__}
```

That's it, the solution is really simple even though the algorithm might have looked frightening.  
The combination *full control over permutation + key whitening* was all that was needed to bring down the cipher with 384 carefully chosen keys.

Thank to problem setter **ph1ll** for a nice chall and the **hacklu team** for the ctf.

### Resources

Pseudo code for the server:
```python
# key is regenerated at each connection
send(generate_challenge())
while True:
  input = recv_input()
  send(generate_keystream_block(input))
```


For the missing details, like how to compute the input to obtain the desired permutation, there is a [python code on github](https://github.com/unjambonakap/ctf/blob/master/hacklu/18/crypto/escape_the_grip/main.py).  
 Warning though, it won't run without setting up tons of dependencies :) (see other repos).

