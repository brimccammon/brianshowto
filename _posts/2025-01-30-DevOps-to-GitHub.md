---
layout: post
title: Azure DevOps Repo to GitHub
---

I currently have sevral private repos on Azure DevOps that I want to move to GitHub.  The easist way I could find to do it is to mirror the DevOps repo to my machine and then push it to GitHub.  To do this either make a directory or change directory to where you want the repo files to be.

''''
mkdir ~/GitTransfer
cd ~/GitTransfer
''''

After the directory has been made and changed to it, mirror the DevOps repo.  I access my repos with SSH so I use the SSH for both DevOps and GitHub.  If doing this be sure to add the key first.

````
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/git-user

git clone --mirror <SSH DevOps URL>
````

This will create a directory for your repo ending in .git.  For example a repo named "Test" would create a directory called "Test.repo"  Change Directory to the repo directory.

````
cd Test.git
````

Next add a remote shortcut for the GitHub repo to move to.

````
git remote add github <SSH GitHub URL>
````

Once the remote shortcut has been added then push the local mirror of the DevOps repo to the new GitHub repo.

````
git push github --all
````

When I did this for one of my repos I got an error because at one point I had uploaded a large ISO to my DevOps repo.  Even though it was deleted the history was still there.   

remote: fatal: pack exceeds maximum allowed size (2.00 GiB)

DevOps allows for large blob storage but GitHub has a hard limit at 2GB.  To find the file I had to run

````
git verify-pack -v objects/pack/*.idx | sort -k3 -n | tail -20
````

This gives the top 20 largest files with the largest at the bottom, in my case it looked like this:

````
065e9b5cfd4ccc75e446b3caad7df93e73a09a74 blob   1978372 773245 92792965 1 37d5c6babf5a2040feef3fc7f5b97e10acb3d12e
9dbf928ccb9394cdd68ee6bc08fd387f8af998c9 blob   2015618 580573 63846253
a18ccc761ce03614650f1ad6a908a443480b46c2 blob   2267763 855316 120249765 2 065e9b5cfd4ccc75e446b3caad7df93e73a09a74
7bf331c2fc0ddb7f9839c87312a83fc24e584bfd blob   3003931 903710 20099092
5c90016ed9c3c2c150bbac1fd818c549d18656a8 blob   3284480 964154 89864529
21371b1cea1bde1424bb380717b2e00d34b55218 blob   3424256 1208354 12766772
09612a22af3fd19767165bf2ace7515339092b37 blob   3571367 253466 74189236
5334c97be6d78aae6c0f3fadf4aa43ddd33a295e blob   3793920 988851 1161913
e0a99de991149555a4c6dc4b4f0461f4aa0c1ee2 blob   4233603 4213562 75660292
210a5b3fa65be4f4c226f48e56c122e8cd435d60 blob   4598784 1381434 118760208
e3c37554753da159beba1ff0006cfd4068427b6b blob   4760064 1340028 91452703
37d5c6babf5a2040feef3fc7f5b97e10acb3d12e blob   4954112 1393799 5934266
7b1205f391440eb56e7c6d0b3d6763c39de96805 blob   4965584 1583638 79958582
8548a4f2edd24fb5127a032ae697e63ef58977cc blob   5185232 1652640 72536596
53241bd1c6e62368839b76f2d5cd292306d28093 blob   5812328 1572636 2582096
4f2f00e5b9d76a825ccd4f4b1560652acf88fc5f blob   9824256 2241841 95668301
b3e575a85a40e42c264b8cf400c8eb0beeef6d8a blob   12961280 3186540 7328065
07c99d5cc89a49ab84ba22a03dae45fca6cf8449 blob   14496025 637491 4154732
412cbe95c80b356d3481aec126197068297a6f52 blob   20437257 20404808 97910464
3d1a29d89fe06d11035c66b9978972a055e1db02 blob   2716784640 2189153983 121189846
````

The last one was about 2.5GB so to find what file that is I had to run

````
git rev-list --objects --all | grep 3d1a29d89fe06d11035c66b9978972a055e1db02
````

That returned

````
3d1a29d89fe06d11035c66b9978972a055e1db02 en_sql_server_2016_standard_with_service_pack_1_x64_dvd_9540929.iso
````

So now I know the offending file.  I had to rewrite the history to remove the old ISO.  I am running Fedora KDE so I needed to install git-filter-repo

````
sudo dnf install git-filter-repo
````

If attempting this on Windows then Python will need to be installed and then git-filter-repo can be installed via pip

````
pip install git-filter-repo
````

Once git-filter-repo was installed I could remove all traces of the ISO from the commit history with the command

````
git filter-repo --path en_sql_server_2016_standard_with_service_pack_1_x64_dvd_9540929.iso --invert-paths --force
````

After that I cleaned up the logs

````
git reflog expire --expire=now --all
````

Finally I was ready to try pushing the local mirror of the DevOps repo to GitHub again.

````
git push github --all
````

Success!!!