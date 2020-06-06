---
title: "Portable Emacs with magit"
date: 2020-06-06
categories: emacs
---

* Requirements
** Hardware
   1. USB or Portable SSD
** Software
   1. Windows OS
   2. GNU Emacs
   3. Portable Git
** File structure
   - emacs-26.3-x86_64
   - portablegit
   - home
   - chanheejeong
     + .git
     + dailylog
     + medicine
     + dictionary
     + documents
   - runemacs.bat

* Initial setup
** GNU Emacs
   1. Download GNU Emacs from http://ftp.gnu.org/gnu/emacs/windows/emacs-26/
      - Download emacs-26.3-x86_64.zip
      - Unzip into portable drive
   2. Make a batch file to run runemacs.exe from portable drive's root directory
      - Default directory of runemacs.exe is /emacs-26.3-x86_64/bin
      - Make runemacs.bat in drive's root directory
        #+BEGIN_SRC
	START \emacs-26.3-x86_64\bin\runemacs.exe
        #+END_SRC
   3. Set HOME directory to be \home inside portable drive
      - Default directory of HOME is C:\Users\...\Appdata\Roaming which is machine-dependent
      - Make site-start.el in emacs-26.3-x86_64\share\emacs\site-lisp
        #+BEGIN_SRC emacs-lisp
	(defvar usb-drive-letter (substring data-directory 0 3))
	(defvar usb-home-dir (concat usb-drive-letter "home/"))
	(setenv "HOME" usb-home-dir)
	#+END_SRC
      - https://at-aka.blogspot.com/2006/06/portable-emacs-22050-on-usb.html
   4. Make a config file to configure emacs
      - Make .emacs in \home
      - Add basic configuration to make emacs usable
        1. Use melpa repository
           #+BEGIN_SRC emacs-lisp
           ;; Use melpa repository
	   (require 'package)
	   (add-to-list 'package-archives '("melpa" . "http://melpa.org/packages/"))
	   (package-initialize)
           #+END_SRC
	2. Remove unnecessary guis
	   #+BEGIN_SRC emacs-lisp
	   ;; Remove unnecessary guis
	   (menu-bar-mode 0)
	   (tool-bar-mode 0)
	   #+END_SRC
	3. Remove splash screen
	   #+BEGIN_SRC emacs-lisp
	   ;; Remove splash screen
	   (setq inhibit-startup-screen t)
           #+END_SRC
	4. Set encoding system
	   #+BEGIN_SRC emacs-lisp
	   ;; Set encoding system
	   (set-language-environment "UTF-8")
	   (set-file-name-coding-system 'cp949-dos)
	   #+END_SRC
	5. Unbind Shift-Space
	   #+BEGIN_SRC emacs-lisp
	   ;; Unbind Shift-Space
	   (global-unset-key (kbd "S-SPC"))
	   #+END_SRC
** More configuration
   1. Backup separately mirroring full path
      #+BEGIN_SRC emacs-lisp
      (defun my-backup-file-name (fpath)
        (let* (
              (backupRootDir "~/backup/")
	      (filePath (replace-regexp-in-string "[A-Za-z]:" "" fpath))
	      (backupFilePath (replace-regexp-in-string "//" "/" (concat backupRootDir filePath "~")))
	      )
	   (make-directory (file-name-directory backupFilePath) (file-name-directory backupFilePath))
	   backupFilePath
	   )
        )
      (setq make-backup-file-name-function 'my-backup-file-name)
      #+END_SRC
      http://ergoemacs.org/emacs/emacs_set_backup_into_a_directory.html
   2. Set theme (leuven)
      No need to install theme from melpa
      #+BEGIN_SRC emacs-lisp
      ;; Set theme (leuven)
      (load-theme 'leuven t)
      #+END_SRC
      https://github.com/fniessen/emacs-leuven-theme
   3. Set font (D2Coding)
      Download and install D2Coding font
      #+BEGIN_SRC emacs-lisp
      ;; Set font (D2Coding)
      (set-face-attribute 'default nil :family "D2Coding")
      (set-face-attribute 'default nil :height 140)
      #+END_SRC
      https://github.com/naver/d2codingfont
   4. (Org-mode) Global key bindings
      #+BEGIN_SRC emacs-lisp
      ;; (Org-mode) Global key bindings
      (global-set-key (kbd "C-c l") 'org-store-link)
      (global-set-key (kbd "C-c a") 'org-agenda)
      (global-set-key (kbd "C-c c") 'org-capture)
      #+END_SRC
** Portable Git & Magit
   1. Download Portable Git from https://git-scm.com/download/win
      - Download PortableGit-2.27.0-64-bit.7z.exe
      - Unzip into portable drive (rename folder to be portablegit)
   2. Set HOME directory to be \home inside portablegit folder
      - Default directory of HOME is /c/Users/...
      - Inside \portablegit\etc\profile, add following code
	#+BEGIN_SRC
	HOME="/home"
	#+END_SRC
   3. Initialize git inside #gitfolderdir
      - Open git-bash.exe and navigate to the folder
        #+BEGIN_SRC
	cd #gitfolderdir
	#+END_SRC
      - Initialize git (git init)
   4. Create a github repository
      - Create a github repository at https://github.com
      - Now the repository is accessible at https://github.com/#username/#reponame
   5. Edit .git config file
      - Inside \#gitfolderdir\.git\config, add following code
	#+BEGIN_SRC
	[user]
	  name = #username
	  email = #useremail@example.com
	[remote "origin"]
	  url = git@github.com:#username/#reponame.git
	  fetch = +refs/heads/*:refs/remotes/origin/*
	[branch "master"]
	  remote = origin
	  merge = refs/heads/master
	#+END_SRC
   6. Generate and add a new SSH key
      - Open git-bash.exe
      - Generate a new SSH key
	#+BEGIN_SRC
	ssh-keygen -t rsa -b 4096 -C "#useremail@example.com"
	#+END_SRC
      - Cut and paste .ssh folder from /c/Users/... to gitportable/home and /home
      - In github webpage, go to Settings > SSH and GPG keys > New SSH keys
      - Copy and paste the contents of id_rsa.pub and save
   7. Pull from github repository
      - Open git-bash.exe and navigate to the folder
	#+BEGIN_SRC
	cd #gitfolderdir
	#+END_SRC
      - Pull from origin master
        #+BEGIN_SRC
        git pull origin master --allow-unrelated-histories
	#+END_SRC
   8. Install and configure Magit
      - Install Magit
	#+BEGIN_SRC emacs-lisp
	M-x package-install RET magit RET
	#+END_SRC
      - Inside .emacs, add following code
	#+BEGIN_SRC emacs-lisp
	;; Magit configuration
	(global-set-key (kbd "C-x g") 'magit-status)
	(defvar git-dir (concat usb-drive-letter "git/bin/git.exe"))
	(setq magit-git-executable git-dir)
	(setenv "SSH_ASKPASS" "git-gui--askpass")
	#+END_SRC
