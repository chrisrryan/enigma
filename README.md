# Enigma Machine Simulation 

Java implementation of an Enigma machine, along with various methods that breaks the encryption. 

An enigma machine is a mechanical encryption device that saw a lot of use during WW2. This code simulates the three rotor enigma which was commonly used. Alan Turing designed machines (Bombes) during the early 1940s at Bletchley Park which broke this cipher.
Seventy years on, using a modern 4GHz 6-core processor, this is still a surprising difficult to break. Even using the detailed knowledge of the machine's workings and insights into the cipher's vulnerabilities it still takes around a minute to decrypt a short message. 

## The Enigma Machine
The code for the enigma machine can be found in the `enigma` package. In the `analysis` package is the code to perform attacks on ciphertext. The attack uses various fitness functions that attempt to measure the effectiveness of a test decryption within the `analysis.fitness` package. `Main.java` file is where the cipher attack is executed.

### Creating a Java Enigma
You can create a new enigma machine using a constructor, for example this code will create a new object called `enigmaMachine` with the settings provided:

```java
enigmaMachine = new Enigma(new String[] {"VII", "V", "IV"}, "B", new int[] {10,5,12}, new int[] {1,2,3}, "AD FT WH JO PN");
```

Rotors and the reflector are given by their common names used in the war, with rotors labelled as `"I"` through to `"VIII"`, and reflectors `"B"` and `"C"`.

### Encrypting and Decrypting
Given an enigma instance like the `enigmaMachine` above, encryption or decryption is performed on character arrays of capital letters [A-Z]. Here is an encryption example using the enigma machine above:

```java
char[] plaintext = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".toCharArray();
char[] ciphertext = enigmaMachine.encrypt(plaintext);
String s = new String(ciphertext); // UJFZBOKXBAQSGCLDNUTSNTASEF
```
You can quickly check everything is working by running the tests found in the `EnigmaTest.java` file.

### How it works
Throughout the enigma machine, letters A-Z are represented as integers 0-25. Most of the components, the rotors, reflector and plugboard are treated as arrays that map values 0-25 to a different set of values 0-25. Encrypting or decrypting is simply a case of passing a value through these arrays in turn. What makes enigma trickier is that the arrays rotate, and that they can have different starting or ring positions. For efficiency, the implementation keeps the arrays fixed, and simulate rotation by shifting the index in and out of each rotor. Before each character is encrypted the rotors rotate, sometimes causing the neighbouring rotors to also rotate, this is handled by the `rotate()` function. Enigma has a quirk whereby the middle rotors moves twice when it turns the left-most rotor. This is called double stepping, and is also implemented here.

## Breaking a Code
Breaking an enigma message here comes down to decrypting a ciphertext with all possible rotor configurations and seeing which output looks the best. We measure best here using a fitness function.

### Fitness functions
The code makes a number of fitness functions available that can be used to measure how close a test decryption is to English text. Each works similarly, some work better than others. You can test to see which work best for a given message. The fitness functions are:
* **Index of coincidence**. The probability of any random two letters being identical. Tends to be higher for proper sentences than for random encrypted text. I've found this is quite good as an initial fitness function when there are many plugs involved.
* **Single / Bi / Tri / Quad grams**. The probability of a sentence measured based on the probability of constituent sequences of characters. Bigrams are pairs, such as AA or ST. Trigrams are triplets, e.g. THE, and so on. The more letters you use, e.g. single -> bi -> tri -> quad seems to improve the power of the fitness function, but you can't rely on this. I've found the longer measures are better when you already have some of the settings correct.
* **Plaintext Fitness**. This function is a known plaintext attack, comparing the decryption against all or portions of a suspected real plaintext. This is by far the most effective solution, even a few words of known plaintext will substantially increase your odds of a break even with a number of plugboard swaps. The constructor for this fitness function has two possible constructors:
```java
public KnownPlaintextFitness(char[] plaintext)
```
Use this if you have an entire complete plaintext you're looking for.

```java
public KnownPlaintextFitness(String[] words, int[] offsets)
```
This one takes pairs of words and their possible positions within the plaintext. For example, in the string "tobeornottobethatisthequestion" you might supply {"to", "that", "question"} and {0, 13, 22}. This function is used if you can guess some words, but aren't sure of the whole sentence, such as when you have partially broken the message already. Note that in the default example known plaintext won't improve much, because the attack is already successful. The errors in the output are not due to the fitness function, rather that we are not simultaneously pairing rotors and ring settings.

### Ciphertext Analysis
The basic approach to the attack is as follows:
1. Decrypt the ciphertext with every possible rotor in each position, and rotated to each starting position. All rotor ring settings are set to 0. No plugboard. For each decryption, measure the text fitness using one of the available fitness functions. Save the best performing rotor configuration.
2. Fix the rotors, and iterate through all possible ring settings for the middle and right rotors, again testing the fitness of the decryption. You do not have to use the same fitness function as before.
3. Fix all settings, and then use a hill climbing approach to find the best performing plugboard swaps, again measured using a fitness function.

## Resources
1. For more details on Enigma, the [wikipedia articles](https://en.wikipedia.org/wiki/Enigma_machine).

2. The code is a development of code by [Dr Mike Pound University of Nottingham](https://www.nottingham.ac.uk/research/beacons-of-excellence/future-food/meet-the-team/michael-pound/index.aspx)

3. The attack used to crack the cipher is based upon work by James Gillogly. [Details](https://web.archive.org/web/20060720040135/http://members.fortunecity.com/jpeschel/gillog1.htm).

