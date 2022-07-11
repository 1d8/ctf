# TryHackMe's Mr. Phisher - https://tryhackme.com/room/mrphisher

This challenge involves deobfuscating macros in a Microsoft Office Document to discover our flag!

We're given an Ubuntu machine which has a document named `MrPhisher.docm`. Since it has the `.docm` extension, we can assume that it does indeed contain macros. I chose to download this file to my Remnux machine so that way I have access to the analysis tools that Remnux offers such as **olevba** which will help us extract the macros without having to open the document.

To extract the macros using **olevba**, run: `olevba MrPhisher.docm` which outputs:

![](https://i.imgur.com/EatTLqw.png)

Macros in text form:

```visual-basic
Sub Format()
    Dim a()
    Dim b As String
    a = Array(102, 109, 99, 100, 127, 100, 53, 62, 105, 57, 61, 106, 62, 62, 55, 110, 113, 114, 118, 39, 36, 118, 47, 35, 32, 125, 34, 46, 46, 124, 43, 124, 25, 71, 26, 71, 21, 88)
    For i = 0 To UBound(a)
        b = b & Chr(a(i) Xor i)
        Next
End Sub
```

Let's perform some code analysis!

```visual-basic
Sub Format()
    Dim a()
    Dim b As String
```

First off, the function is started as indicated by the `Sub` keyword, then two variables are defined: `a` which is an array and `b` which is a string.

```visual-basic
a = Array(102, 109, 99, 100, 127, 100, 53, 62, 105, 57, 61, 106, 62, 62, 55, 110, 113, 114, 118, 39, 36, 118, 47, 35, 32, 125, 34, 46, 46, 124, 43, 124, 25, 71, 26, 71, 21, 88)
For i = 0 To UBound(a)
    b = b & Chr(a(i) Xor i)
    Next
```

Now we fill that array with a sequence of numbers that we can assume are ASCII character codes! But this isn't as simple as a quick conversion from the character code to its string equivalent.

Then we loop through each item in the array, defining the variable `i` as our counter. And for each iteration, we xor the array item at the `i` index with the integer value of `i`, then we run `Chr()` on the result which turns it into its string equivalent, then we add that deobfuscated string value to the variable `b`. This continues for each item in the array until all items have been deobfuscated!

We can run this using Word and simply add in `MsgBox b` in order to display the final output after it's all deobfuscated, but I chose to recreate this in Python!

The xor operation in Python is indicated by the `^` symbol.

```python
obfData = [102, 109, 99, 100, 127, 100, 53, 62, 105, 57, 61, 106, 62, 62, 55, 110, 113, 114, 118, 39, 36, 118, 47, 35, 32, 125, 34, 46, 46, 124, 43, 124, 25, 71, 26, 71, 21, 88]

deobfStr = ""
for i in range(len(obfData)):
    deobfStr += chr(obfData[i] ^ i)

print(deobfStr)
```

As you can see, we're doing the exact same thing as the VBA script. We're looping through each item in the list, then xor'ing the item by the `i` value which is what iteration we are currently on. Then we're turning that back into a string character using `chr()`, then adding that value to `deobfStr`.

Running this will output our flag!

## References

* https://zeltser.com/analyzing-malicious-documents/
* https://codesource.io/how-to-use-the-xor-operator-in-python/
