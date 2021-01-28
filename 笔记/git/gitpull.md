```
mkdir -p $GO_PROJECT_FOLDER && git clone #{gita} $GO_PROJECT_FOLDER && cd $GO_PROJECT_FOLDER
if [[ -n $branch && -z $tag ]]
then       
        echo "branch is $branch and tag is $tag"
        echo "start branch..."
        git fetch
        git checkout $branch
        git pull
        touch  .buildinfo
        git rev-parse HEAD >> .buildinfo
        git rev-parse --abbrev-ref HEAD >> .buildinfo
        git branch -a
        echo "print git info:"
        cat .buildinfo
       if [[ $? -ne 0 ]]
       then
                echo "git checkout version error"
                exit 250
       fi
```
```
elif [[ -n $tag && -z $branch ]]
then
        echo "branch is $branch and tag is $tag"
        echo "starting tag"
        git fetch
        git checkout $tag
        touch  .buildinfo
        git rev-parse HEAD >> .buildinfo
        git rev-parse --abbrev-ref HEAD >> .buildinfo
        git branch -a
        echo "print git info:"
        cat .buildinfo
        echo "checkout done"
       if [[ $? -ne 0 ]]
       then
                echo "git checkout version error"
                exit 250
       fi
else
                echo "git checkout version error"
                exit 250
fi
```