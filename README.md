# How to manually calculate bip39

The goal is to calculate BIP39 12 words mnemonic starting from an entropy. In this case let's suppose we already have a 128bit entropy that means 16 bytes, 32 hex chars. for example

```
656d338db7c217ad57b2516cdddd6d06
```

check hex length
```
massmux@augustus:~$ echo -n 656d338db7c217ad57b2516cdddd6d06  |wc -c
32
```

convert in binary and calculate SHA256 HASH

```
massmux@augustus:~$ printf 656d338db7c217ad57b2516cdddd6d06 | xxd -r -p | sha256sum -b  
bfa0c1bdb5469d1fd77ec004acf783f0b7183ace420f286c15411c11c463c9eb *-
```
Now let's take the first 4 bits (ie 1 hex char) of the resulting hash. This is the checksum. In our case it is _b_

Now let's add this to the initial entropy at the end. So we get
```
656d338db7c217ad57b2516cdddd6d06b
```

Now let's change in binary
```
massmux@augustus:~$ echo "ibase=16; obase=2; $(echo 656d338db7c217ad57b2516cdddd6d06b | tr '[:lower:]' '[:upper:]') " | bc |tr -d '\n'
11001010110110100110011100011011011011111000010000101111010110101010\111101100100101000101101100110111011101110101101101000001101011
```
We must divide the binary output we got above in 12 segments of 11 bits each. Each of them is a word

```
massmux@augustus:~$ printf 11001010110110100110011100011011011011111000010000101111010110101010\111101100100101000101101100110111011101110101101101000001101011 | sed 's/.\{11\}/& /g' | tr " " "\n"
11001010110
11010011001
11000110110
11011111000
01000010111
10101101010
10111101100
10010100010
11011001101
11011101110
10110110100
0001101011
```

We are now noticing that the segments are not omogeneized. We must omogeneize them. For such purpose just add at the beginning of the binary a number of zeros which is the same as the remaining  digits of last segment. In this case we miss 1 digit. So we add 1 zero at the beginning of the binary above and we redo the calculation

```
massmux@augustus:~$ gustus:~$ printf 011001010110110100110011100011011011011111000010000101111010110101010\111101100100101000101101100110111011101110101101101000001101011 | sed 's/.\{11\}/& /g' | tr " " "\n"
01100101011
01101001100
11100011011
01101111100
00100001011
11010110101
01011110110
01001010001
01101100110
11101110111
01011011010
00001101011
```
Now we got 12 perfect 11 bits segments. Each is a bip39 word. Now we have to convert them. In this way

```
massmux@augustus:~$ echo "ibase=2; obase=A; 01100101011" | bc 
811
```
check the table: https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt but consider that the table starts from 1 while we start from zero. So we have to take the next one on the table. in this case we have _grace_

Now we go on with second word:

```
massmux@augustus:~$ echo "ibase=2; obase=A; 01101001100" | bc
844
```
we get the word _hat_

and so on

