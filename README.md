## How to elegantly maintain a patched `dwm` in Archlinux

### Set-up

Get the PKGBUILD:

    mkdir ~/build
    git clone http://aur4.archlinux.org/dwm-git ~/build/dwm-git
    cd ~/build/dwm-git

Grab and extract the source:

    makepkg -o

Enter the `dwm` git repository that `makepkg` grabbed:

    cd src/dwm

Pull my patched `dwm` branch:

    git remote add qguv https://github.com/qguv/dwm
    git pull qguv patchset
    git checkout patchset

Alternatively, create your own patches in the `patchset` branch. You can push this branch to a remote to share your patches to the git-enlightened. Just make sure to checkout your patched code before moving on; it's the contents of the `src/dwm` directory that get installed.

Get rid of `config.h` if you've built `dwm` before. It prevents `config.def.h` from being copied over to the build.

    rm -f config.h

Now tell `makepkg` to build and install the contents of `src/dwm`:

    cd ~/build/dwm-git
    makepkg --noextract -irs

The `--noextract` option is needed to prevent `makepkg` from changing the current state of `src/dwm` in any way (e.g. checking out the latest HEAD).

### Keeping patches up to date

Get the latest code:

    cd ~/build/dwm-git/src/dwm
    git pull origin master
    git pull --force qguv patchset

Choose a newer point in `dwm` history on which to base the patches. Some good choices are:

  - the latest tag (e.g. 6.1)
  - the latest head (e.g. master)
  - the tag for the dwm version in the official arch source tree (e.g. 6.0)

Rebase my patches to the chosen rebase point:

    git checkout patchset
    git rebase "$rebase_point"

Resolve conflicts when they come up, as you would with any rebase.

### Saving patches for gitless peasents

If you're updating the patchset to a particular tag, it may be a good idea to save the individual commits as manual patches for those (poor souls) who don't use git. You can post this on the `dwm` page, but you may want to link to your git-maintained patchset too.

    git diff "$rebase_point" > "~/patches/${USER}s-dwm-$rebase_point.patch"
