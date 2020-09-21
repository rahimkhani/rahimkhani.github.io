Title: Installing Qutip 
Date: 2020-01-09 06:15:00
Modified: 2020-01-09 06:15:00
Category: Quantum
Tags: blog, quantum, qutip, python, pip
Slug: qutip-install
Summary: An Introduction to Qutip and how to install

Qutip is a toolbox of python that used to simulate quantum system. In the past, 
I used to simulate with  Matlab but now I am trying to learn pythonto simulate past works and new works.

I install Octave instead of Matlab in Debian that use past works.

how to install `Python3` and `Qutip`?

I installed `python3` and `pip3`. pip is a tool that download and install python libraries and tools. With `pip3` I can install different toolbox for quantum systems like Qutip.


```sh
sudo apt install python3
sudo apt install pip3
```

To search for qutip packages in pip, one can run the following command:

```sh
pip3 search qutip
```

The output indicates that qutip is already installed on my system:

```
qutip (4.4.1)             - QuTiP: The Quantum Toolbox in Python
  INSTALLED: 4.4.1 (latest)
```

But to install the qutip toolbox and it's dependency packages, run the following command:

```sh
sudo apt install python3-numpy python3-scipy python3-matplotlib cython3
sudo pip3 install qutip
```

After successfully installing the qutip, I can import it's functionality in python by the following snippet.

```python
from qutip import *
```
for write a sample code we import three libraray 
```
import matplotlib.pyplot as plt
import numpy as np
from qutip import *
```
How to define p


The hermitian conjugate
```
sy.dag()
```
The trace
```
H.tr()
```
How to define Hamiltonian 