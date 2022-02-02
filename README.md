# the hitchhiker's guide to the Segfault

The goal of this repository is to make you know what are the extreme cases that can be tested, in order to handle them and have a complete and perfect project

keep in mind that most of the segfaults that you will find are basic ones, as an example, not checking the count of arguments (argc) very well, therefore you can give less arguments than expected which results in a segfault in general, or unprotected malloc functions (i've seen a trend of people protecting the malloc but not checking the return of the function, in this case, the program will also segfault since the program will just start working with NULL, check your mallocs very well and keep track of the return, or use exit() when the allocation fails **if the project allows to**)...

that being said, please read the code carefully and pay attention to details, thats how you find holes in the code :3

i will be ignoring the first 3 42 projects (libft, gnl, printf) in this Readme, as most of the people validated them already, the excess of checkers for those projects out there, and the simplicity of those projects...

i will also be updating reguralry this readme as i am doing more projects and learning/exploring more about them.

if you have any suggestions, ideas, or anything you would want to talk to me about, feel free to contact me on discord : Ophen#3689

a lot of segfaults methods will look a lot alike for multiple projects that take input / read maps, these are the ones i encoutred the most during evaluations / code reviews with friends

buckle up, lets go !

## so_long and FDF:
so_long is a 2D game project, that **TAKES A MAP AS AN ARGUMENT** and renders it on a new window where you can play, in here, there is a lot of things to exploit...
to read the file, we need to open it, right ?, what if we give it a file that doesnt exist, or that doesnt have permission to ?, what if we give an empty map or a maps that has a bunch of null characters inside it ?
here are few commands you should try: 
`./so_long` (if number of arguments isnt checked)
`./so_long ""` (the name of the file has to be checked if it ends with .ber before its openned)
`touch map.ber && chmod 0 map.ber && ./so_long map.ber` (the open function will return -1 as an fd)
`cp /dev/null map1.ber && ./so_long map1.ber` (get_next_line will return null from the start)
`echo > map.ber && ./so_long map.ber` (file with only a new line)
the file contents has to be checked to see if the map is valid, feel free to try a map with open walls, two players, incomplete maps, maps with new lines at the start or at the end or even in between lines, feel free to explore and experiment with the content of the maps :D
you can try the last tests on **fdf** too, and you can also try these : 
in fdf, if we are given a color in the map EX : *'10,0xFF0000'* (RED point of height 10) we have to display the point in that color, if we are not given a color like *'10'*, we default to white, many people use *strchr* with the character *','* and checks if it returns NULL to default to white, or then convert the HEX to an int to display it as a color, this can also be exploited by giving in the map a number with a comma in it but no HEX number, like this *'10,'*, a lot of people will work on the eight character after a comma, which is in this case, beyond the '\0' and results in a segfault.
also feel free to try map with a spaces only, new lines only, a mix of both, etc...

## philosophers
as always try empty strings as an input whenever you can, more or less arguments than expected...
the philosopher program takes 4 to 5 arguments that are, number of philosophers, time to die, time to eat, and time to sleep, try to give it 0 or negative numbers, or even strings (will be converted to 0 if atoi is used), if those parameters go through and the program doesnt check them, it will segfault.
EXAMPLE : trying to allocate a negative number will force the allocation to fail, if the number of philosophers is included in the malloc function and it is negative, and the return value goes unchecked, the program will segfault

## pipex and minishell
as always try empty strings as an input whenever you can, more or less arguments than expected, a non existing infile, or remove the permission for the infile and outfile with chmod 0...
note that the following segfault also works on **minishell** since it uses the envirement variables too :
`env -i ./pipex infile ls cat outfile` or `env -i ./minishell` (starts the program with an empty envirement, which if it is not checked, the program segfaults)
in minishell, there is a lot of cases to explore, since you can give it an infinite number of inputs and therefore possibilities, i personally found over 15 segfaults in some minishell projects in total, i will here show you those i encountred the most :
the easiest segfaults to exploit are in the built-in commands, lets try with **cd** :
`mkdir test`

`cd test`

`rm -rf ../test`

`cd .`

`pwd or ls or anything`(you can also try 'cd .' again, i've seen it segfaults once)

(getcwd will return NULL in this case as the current directory doesnt exist anymore, if this goes unchecked, the program will segfault)
another built-in commands in minishell that are easy to exploit, are the **unset** and **export** commands as they control the envirement variables and take arguments
`export ""`

`unset ""`

`export test=`

`export =test`

`export =`

`export "==="`

`export '= = ='` 

you get it, try as much inputs that doesnt make sense as you want, i ve seen this segfaults many minishell projects...
you can also try and pass an empty string as a command like this `""`

## PUSH_SWAP
as always try empty strings as an input whenever you can, more or less arguments than expected...
in push swap, there isnt much to segfault to be honest, feel free to try "5-" or "--5" or '-' or even '- -' as arguments, this rarely works but you never know :P


------------------------------
well that's it for now, as i said earlier, most of the segfaults you'll ever find in projects are the one you're going to have to read the code pretty well for, i'll say it again and again, check every single allocation if it is protected or not because not everyone keeps track of the return of the function that allocates, i will be updating this readme when i do future projects / find more ways to segfault older projects.
