# Keyboard decoding

The keyboard is divided into 10 rows and 8 columns. The first 8 rows and all the columns are managed by port `0x01` - writing a value to this port sets the row pins (bit 0 corresponds to row 0, and so on) and reading the port reads back the column pins. The remaining two rows are drived by pins 0 and 1 of port `0x02`.

To check which keys are pressed, you need to set the corresponding row pin to 0 and all other rows to 1, then read back the column value. Unfortunately, the column pins take some time to go back from 0 to 1; if you change the selected row and immediately read back the columns, there's a good chance that you will read back the value for the previous row. The original hardware deals with it as follows: immediately upon reading the column value, it switches to the next row and only then processes the read data for the previous row; in the time needed to check all the column bits, hopefully the values will stabilize. From my experience, it is not always sufficient with the new keyboard, especially the joypad keys take a long time to stabilize. More research is needed.

A neat trick to speed up keyboard decoding is to set *all* rows to 0. If the columns read back as `0xFF`, this means that no key is pressed and the whole keyboard scan routine can be skipped.

Please note that the "Power" key is special - it is not present in the keyboard matrix, but instead can be read back as bit 4 in port `0x09`.

## Keyboard matrix (old keyboards)

The original DET1 machines (small, square black or white cases) use the following keyboard matrix:

|		|	Col 0	|	Col 1	|	Col 2	|	Col 3	|	Col 4	|	Col 5	|	Col 6	|	Col 7	|
|	---	|	---	|	---	|	---	|	---	|	---	|	---	|	---	|	---	|
|	**Row 0**	|	Main Menu	|	Back	|	Print	|	F1	|	F2	|	F3	|	F4	|	F5	|
|	**Row 1**	|		|		|		|	@	|	Size	|	Check Spelling	|	Get E-Mail	|	PgUp / Prev	|
|	**Row 2**	|	`	|	1	|	2	|	3	|	4	|	5	|	6	|	7	|
|	**Row 3**	|	8	|	9	|	0	|	-	|	=	|	Delete	|	\	|	PgDn / Next	|
|	**Row 4**	|	Tab	|	Q	|	W	|	E	|	R	|	T	|	Y	|	U	|
|	**Row 5**	|	I	|	O	|	Print	|	[	|	]	|	;	|		|	Enter	|
|	**Row 6**	|	Caps Lock	|	A	|	S	|	D	|	F2	|	G	|	H	|	J	|
|	**Row 7**	|	K	|	L	|	,	|	.	|	/	|	Up	|	Down	|	Right / End	|
|	**Row 8**	|	Left Shift	|	Z	|	X	|	Col 3	|	V	|	B	|	N	|	M	|
|	**Row 9**	|	Function	|		|		|	Space	|		|		|	Right Shift	|	Left / Home	|

## Keyboard matrix (new keyboards)

The new DET2 and DET1D machines (larger, round cases in various shades of blue) use the following keyboard matrix:

|		|	Col 0	|	Col 1	|	Col 2	|	Col 3	|	Col 4	|	Col 5	|	Col 6	|	Col 7	|
|	---	|	---	|	---	|	---	|	---	|	---	|	---	|	---	|	---	|
|	**Row 0**	|	Joypad Up	|	Joypad Right	|	Spelling	|	Get Mail	|	F4	|	F2	|	Back	|	Print	|
|	**Row 1**	|	Joypad Left	|	Joypad Down	|	Select	|	F5	|	F3	|	F1	|	Main Menu	|		|
|	**Row 2**	|	`	|	1	|	2	|	3	|	4	|	5	|	6	|	7	|
|	**Row 3**	|	8	|	9	|	0	|	-	|	=	|	Del	|	PgUp	|	\	|
|	**Row 4**	|	Tab	|	Q	|	W	|	E	|	R	|	T	|	Y	|	U	|
|	**Row 5**	|	I	|	O	|	P	|	[	|	]	|	;	|	'	|	PgDn	|
|	**Row 6**	|	Caps Lock	|	A	|	S	|	D	|	F	|	G	|	H	|	J	|
|	**Row 7**	|	K	|	L	|	,	|	.	|	/	|	Up	|	Down	|	Enter	|
|	**Row 8**	|	Right Shift	|	Z	|	X	|	C	|	V	|	B	|	N	|	M	|
|	**Row 9**	|	Function	|	Size	|		|	Space	|	@	|	Left Shift	|	Left / Home	|	Right / End	|
