# the hitchhiker's guide to the Segfault
------------------------------

The goal of this repository is to make you know what are the extreme cases that can be tested, in order to handle them and have a complete and perfect project && learn more about error handling and taking input / parse data the right way

keep in mind that most of the segfaults that you will find are basic ones, as an example, not checking the count of arguments (argc) very well, therefore you can give fewer arguments than expected which results in a segfault in general, or unprotected malloc functions (I've seen a trend of people protecting the malloc but not checking the return of the function, in this case, the program will also segfault since it will just start working with NULL, check your mallocs very well and keep track of the return, or use exit() when the allocation fails **if the project allows to**)...

That being said, please read the code carefully and pay attention to details, that's how you find holes in the code :3

I will also be updating regularly this README as I am doing more projects and learning/exploring more about them.

If you have any suggestions, ideas, or anything you would want to talk to me about, feel free to contact me on discord : Ophen#3689

a lot of segfaults methods will look a lot alike for multiple projects that take input / read maps, these are the ones I encountered the most during evaluations / code reviews with friends

buckle up, let's go !

## cub3d :
man there is a lot of things that can go wrong in this project, as always, use more or less arguments than expected,test the empty string as an argument, like `./cub3d ""`, an empty `map.cub` file, or a file you don't have permission to...
since cub3d takes a map file as argument, there is a lot of input possibilities that can crash the code successfully, for example giving duplicate values, not giving a value, giving a texture file that doesnt exist, and most importantly, a segfault that few people know about and that i found a lot of times, is giving an invalid XPM file as a texture, (you can create it with `touch test.xpm`, its going to be empty and therefore invalid), the mlx_xpm_file_to_img function will place a Null pointer in our void \*img which will result in a segfault if the value goes unchecked...
feel free to play around with the colors too, giving only the R value for exmaple `F 30,` will result in a segfault if strchr is used to find the commas for the other values and the result goes unchecked.
once again, 99% of the segfaults you will find are just by reading the code, so please read it carefully, see if the return of memory allocations is always checked etc.
a segfault that is really hard to prove its existence even tho we know it exists is if you cast a ray just between two blocks at the corners at the right angle, the ray will go on for the infinity since no wall is there, example of a map where the problem can happen :
```
1111111
1000W001
11111111
```
if you go at the top right corner and cast a ray in the corner from the right angle, it will certainly segfault, (it is the case for most cub3d projects, no one fixes the problem tho since there's no real way to fixing it plus you'll never encounter this case as it needs to be casted pricesly at a small point with the right angle)
play with the map as you wish, it is your only door to segfault the project, Good luck :D

## so_long and FDF:
So_long is a 2D game project, that **TAKES A MAP AS AN ARGUMENT** and renders it on a new window where you can play, in here, there are a lot of things to exploit...
To read the file, we need to open it, right ?, what if we give it a file that doesn't exist, or that doesn't have permission to ?, what if we give an empty map or a map that has a bunch of null characters inside it ?

Here are few commands you should try: 

`./so_long` (if number of arguments isn't checked, this will result in a segfault)

`./so_long ""` (the name of the file has to be checked if it ends with .ber before it is  opened, this may result in a segfault)

`touch map.ber && chmod 0 map.ber && ./so_long map.ber` (the open function will return -1 as an fd, may result in a segfault if program doesnt stop)

`cp /dev/null map1.ber && ./so_long map1.ber` (get_next_line will return null from the start, may result in a segfault if program doesnt stop)

`echo > map.ber && ./so_long map.ber` (file with only a new line, may result in a segfault)

the file content has to be checked to see if the map is valid, feel free to try a map with open walls, two players, incomplete maps, maps with new lines at the start or at the end or even in between lines, feel free to explore and experiment with the content of the maps :D

you can try the last tests on **fdf** too, and you can also try these : 

In fdf, if we are given a color in the map EX : *'10,0xFF0000'* (RED point of height 10) we have to display the point in that color, if we are not given a color like *'10'*, we default to white, many people use *strchr* with the character *','* and checks if it returns NULL to default to white, or then convert the HEX to an int to display it as a color, this can also be exploited by giving in the map a number with a comma in it but no HEX number, like this *'10,'*, a lot of people will work on the eight character after a comma, which is in this case, beyond the '\0' and results in a segfault.

*NOTE : for this case, Strnstr can be used also to find ",0x" or "0x", in which case you'll have to add "10,0x" to try this exploit, try your best to read the code and find the right vulnerabilities*

Also feel free to try a map with spaces only, new lines only, a mix of both, etc...

## philosophers
as always, try empty strings as an input whenever you can, more or less arguments than expected...

The philosopher program takes 4 to 5 arguments that are, number of philosophers, time to die, time to eat, and time to sleep, try to give it 0 or negative numbers, or even strings (will be converted to 0 if atoi is used), if those parameters go through and the program doesn't check them, it will segfault.

EXAMPLE : trying to allocate a negative number will force the allocation to fail, if the number of philosophers is included in the malloc function, and it is negative, and the return value goes unchecked, the program will segfault

## pipex and minishell
as always, try empty strings as an input whenever you can, more or less arguments than expected, a non-existing infile, or remove the permission for the infile and outfile with chmod 0...

Note that the following segfault also works on **minishell** and **pipex** too since it uses the environment variables too :

`env -i ./pipex infile ls cat outfile` or `env -i ./minishell` (starts the program with an empty environment, which if it is not checked, the program segfaults)

you can also unset the `PATH` variable in bash or zsh and run pipex or minishell (and try few commands such as ls), i've seen this segfault a few times

in minishell, there are a lot of cases to explore, since you can give it an infinite number of inputs and therefore possibilities, I personally found over 15 segfaults in some minishell projects in total, I will here show you those i encountred the most :

the easiest segfaults to exploit are in the built-in commands, let's try with **cd** :

```
mkdir test

cd test

rm -rf ../test

cd .

pwd or ls or anything (you can also try 'cd .' again, I've seen it segfaults once)
```

(getcwd will return NULL in this case as the current directory doesn't exist anymore, if this goes unchecked, the program will segfault)

another built-in commands in minishell that are easy to exploit, are the **unset** and **export** commands as they control the environment variables and take arguments

```
export ""

unset ""

export test=

export =test

export =

export "==="

export '= = ='
```

You get it, try as many inputs that doesn't make sense as you want, i ve seen these segfaults many minishell projects...

You can also try and pass an empty string as a command like this `""`

## PUSH_SWAP, Minitalk, Fract-ol
as always, try empty strings as an input whenever you can, more or less arguments than expected...

In push swap, there isn't much to segfault to be honest, feel free to try `"5-"` or `"--5"` or `'-'` or even `'- -'` as arguments, this rarely works, but you never know :P

in minitalk, it is really hard to crash it since the program barely takes any input and there is zero allocation for most projects I've seen, as always, feel free to try empty strings as arguments, more or less parameters than expected, a string instead of the pid for the client etc...

Same thing goes for fract-ol, a project that takes one argument, you can hit it with the classic empty string and wrong number of arguments, but besides that, i m pretty sure you will find quite few unprotected mallocs there that you could exploit

## get_next_line 

compile with the max BUFFER_SIZE you can, aim for the max value the compiler allows you to, it will either segfault or abort 90% of the time if buffer_size value goes unchecked

------------------------------
Well that's it for now, as I said earlier, most of the segfaults you'll ever find in projects are the ones you're going to have to read the code pretty well for, I'll say it again and again, check every single allocation if it is protected or not because not everyone keeps track of the return of the function that allocates, I will be updating this README when I do future projects / find more ways to segfault older projects.
