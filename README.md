# How to manually calculate bip39

The goal is to calculate BIP39 12 words mnemonic starting from an entropy. In this case let's suppose we already have a 128bit entropy that means 16 bytes, 32 hex chars. We just use terminal commands, nothing more. We can use the any offline computer for this calculation. 

### 128 bits entropy

Let's start with a 128bit entropy as follows:

```
656d338db7c217ad57b2516cdddd6d06
```

Checking hexadecimal length

```
echo -n 656d338db7c217ad57b2516cdddd6d06  |wc -c
32
```

### Calculating checksum

Converting data to binary and calculate SHA256 HASH

```
printf 656d338db7c217ad57b2516cdddd6d06 | xxd -r -p | sha256sum -b  
bfa0c1bdb5469d1fd77ec004acf783f0b7183ace420f286c15411c11c463c9eb *-
```

Now let's take the first 4 bits (ie 1 hex char) of the resulting hash. This is the *checksum*. In our case it is _b_   We must then add this checksum to the initial provided entropy at the end of the hexadecimal data. So we get

```
656d338db7c217ad57b2516cdddd6d06b
```

### Get the binary

Now let's change the whole hexadecimal to binary

```
echo "ibase=16; obase=2; $(echo 656d338db7c217ad57b2516cdddd6d06b | tr '[:lower:]' '[:upper:]') " | bc |tr -d '\n'
11001010110110100110011100011011011011111000010000101111010110101010\111101100100101000101101100110111011101110101101101000001101011
```

### Divide in segments

We must divide the binary output we got above, in 12 segments of 11 bits each. Each of them is a word. Infact we expect to reach 12 words to check against a special dictionary. So we do:

```
printf 11001010110110100110011100011011011011111000010000101111010110101010\111101100100101000101101100110111011101110101101101000001101011 | sed 's/.\{11\}/& /g' | tr " " "\n"
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

We are now noticing that the segments are not omogeneized. We must omogeneize them. For such purpose just add at the beginning of the binary a number of zeros which is the same as the remaining digits of last segment. In this case we miss 1 digit. So we add 1 zero at the beginning of the binary above and we redo the calculation

```
printf 011001010110110100110011100011011011011111000010000101111010110101010\111101100100101000101101100110111011101110101101101000001101011 | sed 's/.\{11\}/& /g' | tr " " "\n"
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

### Get the words sequence

Now we got 12 perfect 11 bits segments. Each is a bip39 word. Now we have to convert each of them

```
echo "ibase=2; obase=A; 01100101011" | bc 
811
```

Checking the BIP39 english words table: https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt . Please note that the table starts from 1 while we start from zero. So we have to take the next one on the table. In this case we have the word _grace_ as you can check yourself

Now we go on with the second word:

```
echo "ibase=2; obase=A; 01101001100" | bc
844
```
and we get the word _hat_

```
echo "ibase=2; obase=A; 11100011011" | bc 
1819
```
(toddler)

```
echo "ibase=2; obase=A; 01101111100" | bc 
892
```
(hurdle)

```
echo "ibase=2; obase=A; 00100001011" | bc 
267
```
(cannon)
```
echo "ibase=2; obase=A; 11010110101" | bc 
1717
```
(stove)
```
echo "ibase=2; obase=A; 01011110110" | bc 
758
```
(gain)

```
echo "ibase=2; obase=A; 01001010001" | bc 
593
```
(enforce)

```
echo "ibase=2; obase=A; 01101100110" | bc 
870
```
(holiday)

```
echo "ibase=2; obase=A; 11101110111" | bc 
1911
```
(upon)

```
echo "ibase=2; obase=A; 01011011010" | bc 
730
```
(forget)

```
echo "ibase=2; obase=A; 00001101011" | bc 
107
```
(aspect)


## Verify the result against ian coleman tool

 You can use the Ian Coleman tool (bip39) to verify all the calculations we did by hand right now. As you can see the whole mnemonic is matching on what is returned by the bip39 tool.

[ian coleman bip39 tool](https://iancoleman.io/bip39/)

![iancoleman-bip39](https://i.ibb.co/QDsC3Cr/bip39-check.png)
