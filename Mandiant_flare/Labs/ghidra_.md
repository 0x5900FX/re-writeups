## Ghidra Lab


1. What is the address of the main() function?


2. What Registry values are being set by the main() function?
What are they being set to?
3. What URL is requested within the main() function and what does it do
with the response?

Reverse engineer the function at address 401290. Note that this function is
called by the main() function. Do not examine function 4011F0 until directed
to do so.
4. Without examining function 4011F0, describe as best you can the
overall logic of this function (401290).
5. Reverse engineer function 4011F0. What does this function do?
Decode the ASCII string data pointed to by the arguments to function 4011F0
found within function 401290. (Hint: Each array element is a pointer to a string.
The first encoded string data occurs at memory address 4130C0. All the
encoded string data is contiguous, and the last encoded value is at address
413199).
6. Describe and/or give an example of the decoded data.