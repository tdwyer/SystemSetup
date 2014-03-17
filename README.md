#!/bin/bash
#### Thomas Dwyer <github@tomd.tel> https://tomd.tel/ GPLv3


## Preform these actions as the Root user


    if [[ ! $UID -eq 0 ] ;then
        echo "This needs to run as Root user to install system configurations"
        echo 1
    fi


## System Installation and Updating Outline


    main() {
        case "${1}" in
            install)
                systemSetup
                gitConfigs
                configureConfigs
                [[ -x $(which apt-get) ]] && cleanDebian
                backupSystemConfigs
                backupUsersConfigs
                installSystemConfigsByLink
                gitBundles
                installBundlesByLink
            ;;
            update)
                updateSystem
                updateVimBundles
            ;;
            *)
                echo "Usage: ${0} [update] [install]"
        esac


## Updateing System


### Update system configs


    updateSystem() {
        repos="/usr/src/vim-etc /usr/src/zsh /usr/src/screen /usr/src/dircolors-jellybeans"
        for repo in repos ;do
            git -C ${repo} pull
        done
    }


### Update vim bundles


    updateVimBundles() {
        git -C /usr/src/vim/src submodule foreach git pull origin master
    }


## System Prep and Setup


    systemSetup() {
        if [[ -x $(which apt-get) ]] ;then
            apt-get install vim-nox screen zsh
        elif [[ -x $(which pacman) ]] ;then
            echo "Install screen-git from AUR"
            pacman -S vim zsh
        else
            echo "This system dose not seem to be Debian or Archlinux"
            echo "This system will requre more manual setup"
            exit 255
        fi
    }


### Remove control over Vim configs from Debian


    cleanDebian() {
        links="/usr/bin/vim /usr/bin/vimdiff /usr/bin/rvim /usr/bin/rview /usr/bin/vi /usr/bin/view /usr/bin/ex"
        for link in links ;do
            rm ${link}
            ln -s /usr/bin/vim.nox /usr/bin/vim
        done
    }


### Backup current configs


#### Backup any original configs


    backupSystemConfigs() {
        orig-configs="/etc/zsh /etc/zshrc /etc/vim /etc/vimrc /etc/gvimrc /etc/screenrc"
        for config in orig-configs ;do
            mv ${config} ${config}-orig
        done
    }


#### Backup and config users home


    backupUsersConfigs() {
        home-confgs=".vim .vimrc .zshrc .zprofile .screenrc"
        mv /root/${config} /root/${config}-orig
        mkdir -p /root/.vim/{tmp,backups}
        touch /root/.zshrc
        touch /root/.zprofile
        for config in home-configs ;do
            for dir in $(ls /home) ;do
                if [[ ${dir} != "lost+found" ]] ;then
                    mv /home/${dir}/${config} /home/${dir}/${config}-orig
                    mkdir -p /home/${dir}/.vim/{tmp,backups}
                    touch /home/${dir}/.zshrc
                    touch /home/${dir}/.zprofile
                fi
                chown -R ${dir}:${dir} /home/${dir}
            done
        done
    }


## Configure System


### Get configs


    gitConfigs() {
        git -C /usr/src/vim/src clone https://github.com/tdwyer/vimrc
        git -C /usr/src/vim/src clone https://github.com/tdwyer/zshrc
        git -C /usr/src/vim/src clone https://github.com/tdwyer/screenrc
        git -C /usr/src/vim/src clone https://github.com/tdwyer/dircolors-jellybeans
    }


### Configure configs


    configureConfigs() {
        mkdir -p /usr/src/vim/{src,bundle}
        if [[ -x $(which apt-get) ]] ;then
            cp /usr/src/vim-etc/vimrc /usr/src/vim-etc/vimrc.not.tracked
            sed -i 's/archlinux.vim/debian.vim/' /usr/src/vim-etc/vimrc.not.tracked
        fi
    }


### Configure vim bundles


#### Clone repos


    gitBundles() {
        git -C /usr/src/vim/src submodule add https://github.com/nanotech/jellybeans.vim
        git -C /usr/src/vim/src submodule add https://github.com/bling/vim-airline
        git -C /usr/src/vim/src submodule add https://github.com/jamessan/vim-gnupg
        git -C /usr/src/vim/src submodule add https://github.com/plasticboy/vim-markdown
        git -C /usr/src/vim/src submodule add https://github.com/tpope/vim-pathogen
        git -C /usr/src/vim/src submodule add https://github.com/vim-scripts/mediawiki.vim
        git -C /usr/src/vim/src submodule add https://github.com/scrooloose/nerdtree
        git -C /usr/src/vim/src submodule add https://github.com/jistr/vim-nerdtree-tabs
        git -C /usr/src/vim/src submodule add https://github.com/hdima/python-syntax
        git -C /usr/src/vim/src submodule add https://github.com/ervandew/supertab
        git -C /usr/src/vim/src submodule add https://github.com/ciaranm/securemodelines
    }


## Install Configurations


### Symbolic Link Install


    installSystemConfigsByLink() {
        cd /usr/src
        ln -s dircolors-jellybeans dircolors
        ln -s /usr/src/zshrc /etc/zsh
        ln -s /usr/src/vimrc /etc/vim
        ln -s /usr/src/vimrc/vimrc /etc/vimrc
        ln -s /usr/src/vimrc/gvimrc /etc/gvimrc
        ln -s /usr/src/screenrc /etc/screen
        ln -s /usr/src/screenrc/screenrc /etc/screenrc
    }


### Soft link install


    installBundlesByLink() {
        cd /usr/src/vim/bundle
        ln -s ../src/* .
    }


## Lets get to it!


    main ${@}


# vim: set ts=4 sw=4 tw=80 et :
