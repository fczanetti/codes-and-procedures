# Installing Zsh (Z shell)

Zsh or Z shell is a Linux shell like Bash or SH, and Bash is the standard one for Ubuntu. 

The reason of installing ZSH is that it is a very customizable option, it allows us to install some pluggins that improve our development routine and sometimes make us more productive. It also has a lot of built-in themes and resources that we can take advantage of.

We are also installing Oh My Zsh, that is a framework for Zsh (Z shell).

#### July 30, 2024

## Install ZSH

1 - First, confirm that you don't have Zsh already installed:

```
zsh --version
```

2 - If the result was not ```zsh x.x.x``` or something similar, we have to install it.

```
sudo apt install zsh
```

3 - After installing, use the previous command ```zsh --version```  to confirm the installation was successfull. We can also use the command ```cat /etc/shells``` to see our installed shells and confirm Zsh is on the list.

4 - Make Zsh your default shell:

```
chsh -s $(which zsh)
```

5 - Logout and login again to use the new default shell.

6 - Use the following command to make sure it worked. If it did, zsh will be in the result.

```
echo $SHELL
```

## Install Oh My Zsh

Oh My Zsh can be installed using the following command. This is the framework that will allow us to customize our new shell.

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

## .zshrc

The default Ubuntu shell, Bash, has its own configuration file named **.bashrc**, and the configuration file for Zsh is **.zshrc**. This file is usually located in /home/USER/.

If you were previously using Bash you can move the settings and all you added in your .bashrc to your new .zshrc file so both can work in a similar way.

## Plugins

If you want to add some plugins you can just edit the "plugins=()" in your .zshrc file. For example, to add git, aliases and ansible we would set like this:

```
plugins=(git aliases ansible)
```

There are a lot of built-in plugins that you just need to add to plugins=() on this link:
https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins

But some plugins have to be previously installed like **zsh-autosuggestions** and **zsh-syntax-highlighting**. On the following links you can check the instructions on how to install and use them:

- https://github.com/zsh-users/zsh-autosuggestions
- https://github.com/zsh-users/zsh-syntax-highlighting/tree/master

## Themes

You may also want to configure a different theme, and there is a lot to look at. To set a new one, just find the **ZSH_THEME** setting in your **.zshrc** file and add the desired theme to it. For example, if you want to use **robbyrussell** theme just set it like this:

```
ZSH_THEME="robbyrussell"
```

A list of different themes with screenshots can be found here:

- https://github.com/ohmyzsh/ohmyzsh/wiki/Themes

## References

- https://github.com/ohmyzsh/ohmyzsh
- https://github.com/zsh-users/zsh-autosuggestions
- https://github.com/zsh-users/zsh-syntax-highlighting/tree/master
