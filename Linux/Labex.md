The shell= It is basically a program that takes the commands to the operating system to perform the task.
Bash-Bourne again shell
pwd-Print working directory.
this command shows the directory we are in.

cd-Change directory
/actually means the root directory.

cd /home/pete/pictures (it will take us to this pictures diectory)
and to navigate again to another directory which is inside there like hawaii can be applied by- cd hawaii

cd .   - current directory
cd ..   - takes you to the directory above the current one.
cd ~   -this directory defaults to your home directory.
cd -    -this will take you to the previous directory you were just at. like a button it is not like just move one step up to the parent directory rather like jump to the previous one from where you get to this current one.


ls- List directories 
ls-list the current directory contents

ls -a  (shows all the files in the directory ,even the hidden files)

Hidden files actually starts with ( . )
ls -l (shows all the details of the files).

ls -al or ls  -la (shows all files with the details info based on which order you have linked up those)

touch myfile (it will create a new empty files).
*It is also used to change the timestamp or update the timestamp of the file.*


*In Linux the file name actully doesnot matter like if there is file like mypic.jpg but does not mean it need to contain the picture rather their can be a document or pdf .*

file myfile.jpg (to the description of the file.)

cat dfile cfile- (it can actually concate two files also it can actually used to see the contents of a file.)
*but this cat command is not good for viewing large files rather only good for short contents*


less- It is actually a command to view the file's texts in a page manner and can use the keyboard commands to navigate through the less.
*q- used to quit from this mode
page up ,page down using up and down arrow and (g) - for moving the cursor the beginning of the text.
G -to move to the end of the cursor.
/search -you search for specific text inside the text but just preface the word you want to search with (/)
h- if you need a little help about the less*


History commands- 
Just clicking the up arrow of the keyboard can be used to find the previous commands that's just been used.

Also the (!!) symbols can be used to get this last command.

*Another important one is using the ctrl-R. it will show all the previous commands in the reverse manner just use it and navigate though all of them to find out which needs to be chosen and then hit again ctrl-R*
*Another important command is clear which clears all the commands  in the terminal and tab key shows the suggestions for the incomplete commands.
*

cp myfile /home/files  (this will copy this myfile to this given directory)

cp *.jpg /home/myfiles (this actually copies all files with this .jpg extensions to the myfiles directory )

ls *.text (list all the files ending with .txt)

#? matches with exactly one character
ls file?.txt (matches file1.txt, fileA.txt but not file10.txt since it matches only one character)

#[ ] matches with any characters from this given range within the bracket.
ls file[123].txt  (matches file1.txt, file2.txt, file3.txt)

ls file[a-c].txt (matches filea.txt, fileb.txt, filec.txt)
cp -r Pumkin/ ~/Documents (-r ensures to copy the whole directory along with its internal files)

cp -i file.txt ~/Documents ((-i=interactive) so this one avoids the unintentional overwrite any files which has similar name to the new file)


cp -riv Pumpkin/ ~/Documents ( -r → copy directories recursively.

-i → ask before overwriting.

-v → verbose output (shows what’s being copied).)

#mv
mv oldfile newfile (used to rename the oldfile to newfile also can be use to rename directories )

mv file2 /home/pete/Documents (to move a file to a new directory)

mv file1 file2  /somedirectory (move more than one file)

*like cp it can also overwrite anything so we need to use (-i) flag to prompt before overwriting anything*

#mkdir
mkdir (make directory)

mkdir books paintings (to create new directories, here it makes multiple directories same time)

mkdir -p books/hemmingway/favorites (can create subdirectories at the same time with -p(parent flag) )

#rm
rm file1 (remove files)

rm -f file1 (this is used to forcefully remove all the files whether they are write protected or not)

rm -i file (this one gives you a prompt on whether you want to actually remove the files or directories)

rm-r directory (-r flag to remove all the files in the directory and also subdirectory)

rmdir  directory (to remove directory )

#find

find /home -name puppies.jpg (we want to find the file named puppies.jpg)
	find /home -type d -name MyFolder (This command searches **all subdirectories under `/home`** for a directory named **MyFolder** and prints the full path(s) if found.)*One cool thing to note is that find doesn't stop at the directory you are searching; it will look inside any subdirectories that directory may have as well.*


#help

help echo (this command helps to understand other commands and as well as available flags)

#man
man ls (gives page of documentations on other commands)

#whatis
whatis cat (gives brie description about the command).

#alias

alias foobar='ls -la' (now instead of typing ls -la one can type footbar as an alias and get the same output)

unalias footbar (to remove the footbar)

*Keep in mind that this command won't save your alias after reboot, so you'll need to add a permanent alias in: ~/.bashrc*

What is ~/.bashrc?

~ → your home directory (e.g., /home/username/).

.bashrc → a hidden file (because it starts with a dot .).

Together: ~/.bashrc → /home/username/.bashrc

It’s a shell startup script that runs automatically every time you open a new interactive Bash terminal session.

⚡ What It’s Used For

You can customize your shell by adding settings inside .bashrc. Common uses include:

Aliases

alias ll='ls -la'
alias gs='git status'


(Shortcuts for frequently used commands)

Environment Variables

export EDITOR=nano
export PATH=$PATH:~/bin


Custom Prompt (PS1)

export PS1="\u@\h:\w$ "


(Changes how your terminal prompt looks)

Functions

mkcd () { mkdir -p "$1" && cd "$1"; }


(Create and enter a directory in one command)

🔄 Applying Changes

After editing ~/.bashrc, run:

source ~/.bashrc


or

. ~/.bashrc


to apply the changes immediately (without restarting your terminal).

✅ In short: ~/.bashrc is your personal Bash configuration file where you can define aliases, environment variables, and custom settings to make your shell more powerful.

#exit
exit (to go out)
logout()