# Parallelizing fully homomorphic encryption for a cloud environment
###### tags: `IEEE`

## 1. Introduction
In conventional computing, we need data centers with massive computing resources in order to meet an organization’s maximum needs of data processing. However, the computing resources are costly and often largely idle which cause computing resources underutilized.
Thus **cloud computing** took over the place of conventional computing with data centers. An organization can obtain the computing resources from cloud service provider. Instead of purchasing the computing resources, organizations can purchase computing services from cloud providers, such as AWS, Azure or GCP, where it is much cheaper then purchasing the computing resources.
![](https://i.imgur.com/XeCDOh7.png)
**Cloud computing** is not without the issues. Organizationshave concerns in moving their data to a cloud due to the dataprivacy. Possible threats to the data privacy could be from cloud providers’ employees, clients, and network hackers.
:::success
"Homomorphic encryption is a form of encryption that permits users to perform computations on its encrypted data without first decrypting it."
--- cited from wikipedia
:::
The difficilt thing is: most of existing encryption schemes require data to be decrypted for computations, where decrypted data becomes vulnerable. But what if a computations scheme can be performed on encrypted data without decryption, then the security of data would not be a concern at all. **Homomorphic encryption** makes it possible to process encrypted data without decryption whereby the encrypted results can only be decrypted by the client who requests the service.
![](https://i.imgur.com/ft3XgBu.png)


## 2. Homomorphic encryption schemes
Traditional cryptosystem based on the work from Diffie-Hellman key exchange scheme, such as ElGamal and RSA, which is multiplicatively, but not additively homomorphic. The other cryptosystem, Paillier scheme, only allows homomorphic addition of two encrypted data. The algorithm mentioned above are so called **partially homomorphic encryption** schemes, which are not practical for most of applications. A **fully homomorphic encryption** scheme which supports arbitrary computations on data has far more practical use than partially homomorphic encryption.
![](https://i.imgur.com/k1aQMVJ.png)


## 3. Gentry scheme and cloud computing
[Gentry's scheme]([https://](https://crypto.stanford.edu/craig/craig-thesis.pdf)), first plausible construction for a fully homomorphic encryption scheme, supports both addition and multiplication operations on ciphertexts. At first, Gentry’s scheme starts with a **somewhat homomorphic encryption** using **ideal lattices** that can only perform a limited number of homomorphic operations on encrypted data. Gentry then modifies the scheme to a fully homomorphic encryption scheme by adding **bootstrapping procedure** to it.

Since the cloud computing has been widely used for parallel processing of mass data. The algorithm used in this work’s evaluation is based on the implementation presented in a 2011 paper by Gentry and Halevi in 2011, seems to be a perfect fit for the cloud because the tasks of performing mathematical operations can be analyzed, split, and distributed to the nodes in the cloud.
```
Gentry encryption: C = pq+2r+m {p=secrete key, q=large ranNum, r=small noise} 
Gentry decryption: m = (c mod p) mod 2
```
:::success
First, we must check it follow the addictive homomorphic and multiple homomorphic, according to [Gentry proposed scheme](https://crypto.stanford.edu/craig/craig-thesis.pdf):
* addiction:
c1+c2 = (m1+m2)+2(r1+r2)+p(q1+q2)
((c1+c2) mod p) mod 2 = (m1+m2)
* multiplication
c1*c2 = (m1+2r1)(m2+2r2)+p(pq1q2+m1q2+m2q1+2r1q2+2r2q1)
(c1*c2 mod p) mod 2 = ((m1+2r1)(m2+2r2) mod 2) = m1m2

The construction starts from a **somewhat homomorphic encryption scheme**, which is limited to evaluating low-degree polynomials over encrypted data. It is limited because each ciphertext is noisy in some sense, and this noise grows as one adds and multiplies ciphertexts, until ultimately the noise makes the resulting ciphertext indecipherable.

Gentry then shows how to slightly modify this scheme to make it **bootstrappable**, i.e., capable of evaluating its own decryption circuit and then at least one more operation. Finally, he shows that any **bootstrappable somewhat homomorphic encryption** scheme can be converted into a **fully homomorphic encryption** through a recursive self-embedding. 
https://www.iacr.org/archive/crypto2010/62230116/62230116.pdf
For Gentry's "noisy" scheme, the bootstrapping procedure effectively "refreshes" the ciphertext by applying to it the decryption procedure homomorphically, thereby obtaining a new ciphertext that encrypts the same value as before but has lower noise. By "refreshing" the ciphertext periodically whenever the noise grows too large, it is possible to compute an arbitrary number of additions and multiplications without increasing the noise too much.

[In another paper](https://www.iacr.org/archive/crypto2010/62230116/62230116.pdf) issued by Gentry, he based on the security of his scheme assuming hardness of two problems: certain **worst-case problems over ideal lattices**, and the **low-weight subset sum problem**. As the result, the Gentry-Halevi implementation of Gentry's original cryptosystem reported about 30 minutes per basic bit operation in dimension 32768.
![](https://i.imgur.com/FFeXO4I.png)
:::

## 4. Algorithm
The author divided algorithm into 3 parts:
### 4.1 Creating data dependency graph
![](https://i.imgur.com/9aYURNZ.png)

The algorithm shown above describes how to create the data: 
```
dependency graph from the input circuit E.

Line1-3: letting P(Parent) and C(Children) be an empty array.
 
Line4: let D be an associative array, which denote the root node.

Line5-13: main loop iterates through the elements e as index. If index is offset 1, 
          take the inputs of e and look up from D. 

Line14-23: Denote d as a parent of the current node, and the current node as a child of d. Move on to next node.
```
To sum up, for each output, every dependent of owner will be marked as a parent of index, and index will be marked as a child of each dependent. Each output of the current operation will be updated to be owned by the current operations in D.

### 4.2 Finding sub-circuit
![](https://i.imgur.com/gpQ13T5.png)

This is easy than the other, the most stuff above can be shorten in one thing: the dispatcher repeats this process of candidate evaluation until there are no more nodes in candidates, which pend all of the children of c and adds them to an ordered array of nodes.
### 4.3 Blind addition and multiplication
After building up dependency graph, the result should be look like this:
![](https://i.imgur.com/6BJ6dgW.png)

Thus we can do the arithmetic of encrypted data, mentioned at Gentry encryption.


## 5. Evaluation of the algorithm

:::success
Structure & methodology explain
They used binary tree structure to perform cloud parallelized computation. When processing with 8 nodes, the additional 4 nodes were provided as a secondary level, and 2 nodes at third level, then the final root. If processing with 4 nodes, and so on...

Moreover, they test only test three computations on the cloud system for testing. All three computations are using the same set of data which contain 20 random 8-bit integers. 
* The first experiment is to compute the sum of these 20 integers. 
* The second is to compute the vector product of the integers. 
* The third one is to compute the variance of the integers.

The network connection between them was a wireless network, thus it migt effect the speed in some way.
![](https://i.imgur.com/albuuYo.png)
:::
Here is some graph that can tell the story:
![](https://i.imgur.com/yqHJOUC.png)
The extraordinary speed occur in vector product, which is reasonable because it follow the logN as the result. However doesn't make sence for me is sum computation, it only speed up while growing nodes from 1 to 2or4, then go back to slow as nodes grow to 8.

![](https://i.imgur.com/95RxD52.png)
It indeed to small amout of time. But the time was too insignificant to mention, I just ignore it(Me, not the author of this paper). Just to mention, same as table 4, 6, 8.

![](https://i.imgur.com/QFKp88Z.png)
The result similar to the time to find sub-circuits in table 4, there was an increase from 2 to 4 nodes, but a decrease from 4 to 8 nodes. The writer give the explain "jump in level is once again most likely due to the limit in processor availability across threads, leading to execution waiting for processor resources." I give credit on this, because there is no reason for decreasing in time while 4 nodes grow to 8 nodes.

![](https://i.imgur.com/cy2mlY9.png)
The time spent finding sub-circuits was less in the 2, 4, and 8 node trials than the 1 node, of course. But the difference in time remains negligible, so... ignore it.

![](https://i.imgur.com/bkXAKff.png)
There was a jump from the 1 and 2 node level to the 4 and 8 node level and the least number of circuits evaluated was in the 8 node configuration. However, it was taking about double of time but reduce 3 sub-circuit, I don't think it's worthy. 

![](https://i.imgur.com/ybDyQ44.png)
There was once again an increase from the time to find sub-circuits with 1 and 2 nodes to 4 and 8 notes, although the difference in time is negligible. 

![](https://i.imgur.com/NXj4tbg.png)
There was a jump from the 1 and 2 node level to the 4 and 8 node level. The least number of subcircuits evaluated was in the 8 node configuration. And the reason... give up on trying to explain.

## 6. Conclusions
In this paper, author described the problem of data privacy in cloud computing, where bring up FHE to resolve the data privacy in the cloud where the encrypted data can be processed and the encrypted results are returned. Although Gentry’s encryption scheme is fully homomorphic, with slow performance make it impractical to real case. Several methods have been proposed to speed up the performance of FHE schemes, and this paper is one of theme: by using parallel processing to distribute the computing time. And parallel processing for Gentry’s encryption presented in this paper show the results that the **proposed parallel processing of Gentry’s encryption improves the performance better than the computations on a single node.** We can do the quick review on table 3, time on processing vector product and variance decrease obviously. Although sum computation was not ideal faster in this case, it didn't effect the result.



## Reference
[Craig Gentry, Toward Basing Fully Homomorphic Encryptionon Worst-Case Hardness, IBM T.J Watson Research Center](https://www.iacr.org/archive/crypto2010/62230116/62230116.pdf)
[Craig Gentry, A Fully Homomorphic Encryption Scheme, Stanford Ph.D, 2009](https://crypto.stanford.edu/craig/craig-thesis.pdf)
[Craig GentryShai&Halevi, Implementing Gentry's Fully-Homomorphic Encryption Scheme, IBM Research Center, 2011](https://eprint.iacr.org/2010/520.pdf)