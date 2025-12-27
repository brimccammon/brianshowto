---
layout: post
title: Remove Passwords or Other Text from VSTS Git
---

In a perfect world no one would ever store usernames and passwords in code.  We are on in a perfect world and I myself make mistakes and in the past I have created scripts that required passwords in them and I accidentally pushed the code to VSTS with the password entered.  So I had to figure out how to remove that from the history of the file.  I could have deleted the project and created a new one but I would have lost all my history of commits which I did not want to do.  Here is how I found to remove text from commit history.

The guys at BFG Repo-Cleaner (https://rtyley.github.io/bfg-repo-cleaner/) really make this easy.  Basically you make a clone of your project in a temp folder and then you run BFG against your temp clone then you expire the reference logs and then prune them and finally push back to the project.  The steps that I use assume that you have your project name Project, you are storing the git projects at C:\Git, you have downloaded BFG to C:\Git and you have a text file which has all the text you want to replace (one phrase per line) saved to C:\Git\pass.txt   These are the exact steps I took:

1. Open command prompt.
2. Make a new temp directory.
````
mkdir C:\Git\TempProject
````
3. Change to the new directory.
````
cd C:\Git\TempProject
````
4. Clone the project to the directory.
````
git clone –mirror https://yourname.visualstudio.com/_git/Project
````
5. Change to the Git directory.
````
cd C:\Git
````
6. Run BFG with the pass.txt file against the TempProject (replace the BFG version number to match what you downloaded).
````
java -jar bfg-1.12.16.jar –replace-text pass.txt TempProject\Project.git
````
7. Change to the TempProject\Project.git directory.
````
cd C:\Git\TempProject\Project.git
````
8. Expire and prune the reference logs.
````
git reflog expire –expire=now –all && git gc –prune=now –aggressive
````
9. Push the changes back to the project.
````
git push
````

Once you follow those steps you can go look at your commit history and the text you wanted replaced will show as ***REMOVED***.  Now your text is safe and you can delete the temp folder that was created in step 2.
