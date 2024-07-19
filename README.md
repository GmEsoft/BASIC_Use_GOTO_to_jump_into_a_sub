BASIC: Enter a subroutine with GOTO
===================================

Context
-------

I have been challenged by one member of a TRS-80 group on Facebook to demonstrate how it could be possible
to enter a BASIC subroutine using the `GOTO` statement.

My examples are running on the TRS-80 Model I Level II BASIC.

My discussion is based on this original post (sorry I don't see that post anymore, 
so I can't mention the author's name):

> I was lambasted the other day for jumping out of a for loop.  So, I learned this simple trick.
> 
> Let’s say you have a for loop like:
> 
> ```basic
> Forz=1ton
> If a(z)=b(z),gosub33
> Next
> …blah, blah, blah…
> 33'sub to handle condition 
> …the thing, the thing, the thing …
> Return
> ```
> 
> Rather than just jumping out of the for loop, try this instead:
> 
> ```basic
> T=0
> Forz=1ton
> If a(z)=b(z),t=z:z=n
> Next
> Ift<>0,gosub33
> …blah, blah, blah…
> 33'sub to handle condition 
> …the thing, the thing, the thing …
> Return
> ```
> 
> The z=n lets you exit the loop, gracefully.
> 
> And, yes, Virginia on the M1L2 and other machines:
> One can substitute the , for the THEN 
> One can specify a comment with an ' rather than a REM
> 

Indeed we can't simply jump out of the FOR loop using a GOTO, because the FOR stack frame is still present on the stack. 

I made a small example to demonstrate that.

Without the control variable names after the NEXT keyword, we get this, showing that the GOTO statements don't actually exit the loop.
In fact, the NEXT statement at line 70 increments either the J variable or the I variable, depending on the last executed statement:
- If the last executed statement was GOTO 70 at line 40, then the NEXT at line 70 would increment the J variable and iterate back to line 30.
- If the last executed statement was PRINT "..." at line 60, then the NEXT at line 70 would increment the I variable and iterate back to line 20.


```basic
>LIST
10 FOR I=1 TO 2
20   FOR J=1 TO 3
30     PRINT"i =";I;" j =";J
40     IF J=2 THEN PRINT"Jump out":GOTO 70
50   NEXT
60   PRINT"exited FOR J loop"
70 NEXT
80 PRINT"exited FOR I loop"
READY
>RUN
i = 1  j = 1
i = 1  j = 2
Jump out
i = 1  j = 3
exited FOR J loop
i = 2  j = 1
i = 2  j = 2
Jump out
i = 2  j = 3
exited FOR J loop
exited FOR I loop
READY
>
````

We clearly see that after the 'Jump out', J is incremented one more time (by the NEXT statement at line 70), meaning that the FOR J
loop was not really exited.

If we specify the control variables I and J after the NEXT keywords, we get something different, more expectable.

```basic
>LIST
10 FOR I=1 TO 2
20   FOR J=1 TO 3
30     PRINT"i =";I;" j =";J
40     IF J=2 THEN PRINT"Jump out":GOTO 70
50   NEXT J
60   PRINT"exited FOR J loop"
70 NEXT I
80 PRINT"exited FOR I loop"
READY
>RUN
i = 1  j = 1
i = 1  j = 2
Jump out
i = 2  j = 1
i = 2  j = 2
Jump out
exited FOR I loop
READY
>
````

Here, by forcing the NEXT statement at line 70 to increment the variable I, the FOR J stack frame is forced to be discarded, so the FOR J loop is now effectively exited.

We can also demonstrate that a RETURN in a loop effectively breaks the loop, with the following example _without_ the control variables after NEXT, 
where the inner loop is moved to a subroutine. The NEXT statement in line 70 always bumps the I variable, whether the FOR J loop was broken or not.

```basic
>LIST
10 FOR I=1 TO 2
20   GOSUB 100
70 NEXT
80 PRINT"exited FOR I loop"
90 END
100 'FOR J loop subroutine
110 FOR J=1 TO 3
120   PRINT"i =";I;" j =";J
130   IF J=2 THEN PRINT"Break out":RETURN
140 NEXT
150 PRINT"exited FOR J loop"
160 RETURN
READY
>RUN
i = 1  j = 1
i = 1  j = 2
Break out
i = 2  j = 1
i = 2  j = 2
Break out
exited FOR I loop
READY
>
````

The RETURN statement in line 130 effectively discards the FOR J stack frame, and the NEXT at line 70 increments the variable I, not J.

Now let's go back to the original post's examples and see how we can rewrite them.

Version 1: the original code made runnable
------------------------------------------

Modified so that it can run on the TRS-80:
- An 'init' block to initialize the variable N and the arrays A(N) and B(N);
- Some PRINT statements to visualize the processing and the variables;
- an END statement to gracefully end the main program.

```basic
1 'init
2 N=3
3 DIM A(N),B(N)
4 FOR Z=1 TO N
5   A(Z)=Z
6 NEXT
7 B(2)=2
10 'main program
12 FOR Z=1 TO N
13   IF A(Z)=B(Z), GOSUB 33
14 NEXT
16 PRINT"blah, blah, blah... => Z =";Z
17 END '(to avoid ?RG Error)
33 'sub to handle the condition
34 PRINT"the thing, the thing, the thing => Z =";Z
35 RETURN
```

Here, the FOR Z loop is executed 3 times, and Z's value is N+1 at the end of the loop:
```
>run
the thing, the thing, the thing => Z = 2
blah, blah, blah... => Z = 4
READY
>
```

Version 2: the proposed code to gracefully exit the FOR loop
------------------------------------------------------------

Modifications:
- Store the matching value of Z to T and assign N to Z in order to skip the remaining iterations;
- the subroutine GOSUB 33 is called after the loop completion;
- In the subroutine, change the variable Z to T where the matching value of Z was stored.

```basic
1 'init
2 N=3
3 DIM A(N),B(N)
4 FOR Z=1 TO N
5   A(Z)=Z
6 NEXT
7 B(2)=2
10 'main program
11 T=0
12 FOR Z=1 TO N
13   IF A(Z)=B(Z),T=Z:Z=N
14 NEXT
15 IF T>0,GOSUB 33
16 PRINT"blah, blah, blah... => Z =";Z
17 END '(to avoid ?RG Error)
33 'sub to handle the condition
34 PRINT"the thing, the thing, the thing => T =";T
35 RETURN
```

Here, the FOR Z loop is only executed 2 times, and Z's value is N+1 at the end of the loop:
```
>run
the thing, the thing, the thing => T = 2
blah, blah, blah... => Z = 4
READY
>
```


Version 3: another way to gracefully exit the loop
--------------------------------------------------

Modifications from version 1:
- just add the `Z=N` statement after `GOSUB 33` in order to skip the remaining iterations.

```basic
1 'init
2 N=3
3 DIM A(N),B(N)
4 FOR Z=1 TO N
5   A(Z)=Z
6 NEXT
7 B(2)=2
10 'main program
12 FOR Z=1 TO N
13   IF A(Z)=B(Z),GOSUB 33:Z=N
14 NEXT
16 PRINT"blah, blah, blah... => Z =";Z
17 END '(to avoid ?RG Error)
33 'sub to handle the condition
34 PRINT"the thing, the thing, the thing => Z =";Z
35 RETURN
```

Here, the FOR Z loop is only executed 2 times, and Z's value is N+1 at the end of the loop;
and the variable T is not needed:
```
>run
the thing, the thing, the thing => Z = 2
blah, blah, blah... => Z = 4
READY
>
```

Version 4: move the FOR loop to a subroutine
--------------------------------------------

Modifications from version 3:
- the loop is moved to a new subroutine starting at line 20;
- the new subroutine is called by GOSUB 20 at line 11 in the main program;

```basic
1 'init
2 N=3
3 DIM A(N),B(N)
4 FOR Z=1 TO N
5   A(Z)=Z
6 NEXT
7 B(2)=2
10 'main program
11 GOSUB 20
16 PRINT"blah, blah, blah... => Z =";Z
17 END '(to avoid ?RG Error)
20 'sub to handle the loop
21 FOR Z=1 TO N
22   IF A(Z)=B(Z),GOSUB 33:Z=N
23 NEXT
24 RETURN
33 'sub to handle the condition
34 PRINT"the thing, the thing, the thing => Z =";Z
35 RETURN
```

Here, the FOR Z loop is only executed 2 times, and Z's value is N+1 at the end of the loop:
```
>run
the thing, the thing, the thing => Z = 2
blah, blah, blah... => Z = 4
READY
>
```

Version 5: exit the FOR loop using RETURN
-----------------------------------------

Modifications from version 4:
- use RETURN to break the loop, instead of Z=N.

```basic
1 'init
2 N=3
3 DIM A(N),B(N)
4 FOR Z=1 TO N
5   A(Z)=Z
6 NEXT
7 B(2)=2
10 'main program
11 GOSUB 20
16 PRINT"blah, blah, blah... => Z =";Z
17 END '(to avoid ?RG Error)
20 'sub to handle the loop
21 FOR Z=1 TO N
22   IF A(Z)=B(Z),GOSUB 33:RETURN
23 NEXT
24 RETURN
33 'sub to handle the condition
34 PRINT"the thing, the thing, the thing => Z =";Z
35 RETURN
```

Here, the FOR Z loop is only executed 2 times, and Z's value after GOSUB 20 is 2, not 4, because we broke the loop:
```
>run
the thing, the thing, the thing => Z = 2
blah, blah, blah... => Z = 2
READY
>
```

Yes, this works! The RETURN statement not only pops the GOSUB 20 return address from the stack, 
but also _discards_ the FOR Z stack frame on top of the GOSUB 20 stack frame.

To be convinced of that, let's replace the line `17 END` with the next 2 lines, and see what happens:

```basic
17 NEXT
18 END
```

We get this:
```
the thing, the thing, the thing => Z = 2
blah, blah, blah... => Z = 4
NEXT without FOR in 17
READY
```

As expected we get a NEXT without FOR error at line 17. This factually proves that the FOR Z frame is no longer on the stack after the RETURN of the subroutine.

This fact is verified in all implementations of BASIC by Microsoft I tested until now, but also in TRS-80 LEVEL I BASIC.

I'd be delighted if someone could tell me BASIC implementations that don't work this way...


There has been some debate around this, sometimes qualifying this way of doing as 'sloppy', but 
it is a very common practice in the industry, in many languages like C, C++ and Java, even in the standard libraries. 
Just see how Java's `LinkedList.indexOf()` or C++ `std::string::find()` are implemented !

```java
    public int indexOf(Object o) {
        int index = 0;
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null)
                    return index;
                index++;
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item))
                    return index;
                index++;
            }
        }
        return -1;
    }
```

```cpp
	size_type __CLR_OR_THIS_CALL find(const _Elem *_Ptr,
		size_type _Off, size_type _Count) const
		{	// look for [_Ptr, _Ptr + _Count) beginnng at or after _Off
		_DEBUG_POINTER(_Ptr);
		if (_Count == 0 && _Off <= _Mysize)
			return (_Off);	// null string always matches (if inside string)

		size_type _Nm;
		if (_Off < _Mysize && _Count <= (_Nm = _Mysize - _Off))
			{	// room for match, look for it
			const _Elem *_Uptr, *_Vptr;
			for (_Nm -= _Count - 1, _Vptr = _Myptr() + _Off;
				(_Uptr = _Traits::find(_Vptr, _Nm, *_Ptr)) != 0;
				_Nm -= _Uptr - _Vptr + 1, _Vptr = _Uptr + 1)
				if (_Traits::compare(_Uptr, _Ptr, _Count) == 0)
					return (_Uptr - _Myptr());	// found a match
			}

			return (npos);	// no match
		}

```

Version 6: replace GOSUB/RETURN with GOTO
-----------------------------------------

Modifications from version 5:
- as a shortcut, replace GOSUB 33:RETURN with GOTO 33.

```basic
1 'init
2 N=3
3 DIM A(N),B(N)
4 FOR Z=1 TO N
5   A(Z)=Z
6 NEXT
7 B(2)=2
10 'main program
11 GOSUB 20
16 PRINT"blah, blah, blah... => Z =";Z
17 END '(to avoid ?RG Error)
20 'sub to handle the loop
21 FOR Z=1 TO N
22   IF A(Z)=B(Z),GOTO 33
23 NEXT
24 RETURN
33 'sub to handle the condition
34 PRINT"the thing, the thing, the thing => Z =";Z
35 RETURN
```

Here, the FOR Z loop is only executed 2 times, and Z's value is 2, not 4, because we broke the loop:
```
>run
the thing, the thing, the thing => Z = 2
blah, blah, blah... => Z = 2
READY
>
```

Here, by replacing GOSUB 33:RETURN with GOTO 33 we avoid pushing another GOSUB frame on the stack,
the program is a little shorter and the performance is a little better.

**So this example _clearly_ demonstrates how we can jump to a subroutine using a GOTO instruction.**

Again, it is a very common practice in the industry, especially in assembly language programming, where
it is often needed to shorten the object code as much as possible, so it can fit the ROM.

For example, here is how Z-80 routines to output the value of A and HL in hexadecimal are usually implemented:

```asm
DDISA   EQU     33H             ; TRS-80 ROM subroutine to display the character in register A

;-----	Display A in hexadecimal
DHEXA:	PUSH    AF              ; save A
        SRL     A               ; shift the upper nibble of A
        SRL     A               ;   to the right
        SRL     A               ;
        SRL     A               ;
        CALL    DHEXN           ; display the lower nibble in A
        POP     AF              ; restore A
        AND     0FH             ; retain the 4 lower bits
                                ; fall through in the DHEXN routine
DHEXN:  CP      10              ; is a digit in range [0..9] ?
        JR      C,DHEXN1        ; jump if yes
        ADD     A,7             ; adjust for [A..F]
DHEX1:	ADD     A,'0'           ; convert to ASCII
        JP      DDISA           ; Jump to the ROM DDISA subroutine
                                ; no RET needed ...

;-----	Display HL in hexadecimal
DHEXHL: LD      A,H             ; get msb H into A
        CALL    DHEXA           ; display msb
        LD      A,L             ; get lsb L into A
        JR      DHEXA           ; display lsb (JUMP into the DHEXA routine)
                                ; no RET needed ...

        END

```

Thanks for your attention!

_Michel Bernard_
