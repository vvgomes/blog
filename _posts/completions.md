install

```
#!/bin/zsh

set -e
set -o errexit

if [ ! -e bix ]
then
  echo "Please run in root directory of domains"
  exit 1
fi

path_to_bix=`pwd`

mkdir -p $HOME/.zsh-completions
cp ${path_to_bix}/bix_completions/_bix $HOME/.zsh-completions

zshrc=$HOME/.zshrc

if [ ! -e $zshrc ]
then
  echo "$zshrc not found."
  exit 1
fi

echo "export PATH=${path_to_bix}:\$PATH" >> $zshrc
echo 'fpath=($HOME/.zsh-completions $fpath)' >> $zshrc
echo 'autoload -U compinit' >> $zshrc
echo 'compinit' >> $zshrc

echo 'bix completions installed successfuly.'
```
completions

```
#compdef bix

actions=`bix -no-color -h | tail +3 | tail -r | tail +7 | tail -r | sed -E "s/([a-z|_]+)/&:'/" | sed -E "s/[ ]+/ /g" | sed -E "s/$/'/" | sed -E "s/' /'/g"`
_arguments "1:Actions:(($actions -h:'prints help'))"


```
